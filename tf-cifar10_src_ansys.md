[TOC]

# Tensorflow Cifar10 官方代码解析：

## 初始化

### 1. global flags init

```python 
FLAGS = tf.app.flags.FLAGS  #定义FLAGS类
#创建flag的一些内容
#训练的log dir
tf.app.flags.DEFINE_string('train_dir', '/tmp/cifar10_train2',
                           """Directory where to write event logs """
                           """and checkpoint.""")
#最多迭代的次数
tf.app.flags.DEFINE_integer('max_steps', 1000,
                            """Number of batches to run.""")

tf.app.flags.DEFINE_boolean('log_device_placement', False,
                            """Whether to log device placement.""")
tf.app.flags.DEFINE_integer('log_frequency', 10,
                              """How often to log results to the console.""")
 
```

### 2. 初始化全局步骤

```python
global_step=tf.train.get_or_create_global_step()
```



## 数据准备


调用cifar10.distorted_inputs()  # 乱序输入

在cifar10.distorted_inputs()中

调用cifar10_input.distorted_inputs(batch_size=FLAGS.batch_size)  ，这里的batch_size 是默认的128

进入后，又调用内部函数_get_images_labels(batch_size, tfds.Split.Train, distords=True)

这个函数返回：

```python
Returns:  images: Images. 4D tensor of [batch_size, IMAGE_SIZE, IMAGE_SIZE, 3] size. 
        labels: Labels. 1D tensor of [batch_size] size.
```

进入这个函数才是重点的图片预处理。

1. 载入图片。import tensorflow_datasets as tfds

   ```python
   dataset = tfds.load(name='cifar10', split=split)   #split = tfds.Split.Train
   ```

2. 加载图片后，对图片进行裁剪，并且进行映射处理。

   ```python
   dataset = dataset.map(DataPreprocessor(distords), num_parallel_calls=10)  
   # 创建了一个DataPreprocessor对象，并把这个对象穿进去。
    # map函数中会调用Dataprocessor(),这时会进入类中的__call__
   scope = 'data_augmentation' if distords else 'input'  # distords=
   with tf.name_scope(scope):
       #处理成了24X24*3 的图片
     dataset = dataset.map(DataPreprocessor(distords), num_parallel_calls=10) # 创建了一个DataPreprocessor对象，并把这个对象穿进去。
    # map函数中会调用Dataprocessor(),这时会进入类中的__call__
   
     # Dataset is small enough to be fully loaded on memory:
   dataset = dataset.prefetch(-1) # 预取出,batchsize=-1,This is the sentinel for auto-tuning 自动协调的哨兵?
   dataset = dataset.repeat().batch(batch_size)
   iterator = dataset.make_one_shot_iterator()
   images_labels = iterator.get_next()
   images, labels = images_labels['input'], images_labels['target']
   tf.summary.image('images', images)
   return images, labels
       
       
   class DataPreprocessor(object):
     """Applies transformations to dataset record."""
   
     def __init__(self, distords):
       self._distords = distords
   
     def __call__(self, record):
       """Process img for training or eval."""
       img = record['image']
       img = tf.cast(img, tf.float32)  # tf.cast 把img的tensor转换成float32的类型
       if self._distords:  # training
         # Randomly crop a [height, width] section of the image.
         img = tf.random_crop(img, [IMAGE_SIZE, IMAGE_SIZE, 3])
         # Randomly flip the image horizontally. #随机裁剪
         img = tf.image.random_flip_left_right(img) #随机翻转
         # Because these operations are not commutative, consider randomizing
         # the order their operation.      #因为考虑这些命令不是可交换的，所以考虑去随机化他们额操作
         # NOTE: since per_image_standardization zeros the mean and makes
         # the stddev unit, this likely has no effect see tensorflow#1458.  #注意：由于per-image标准化使平均值为零，并使stdev单位为零，因此这可能没有任何影响。
         img = tf.image.random_brightness(img, max_delta=63)
         img = tf.image.random_contrast(img, lower=0.2, upper=1.8)
       else:  # Image processing for evaluation.
         # Crop the central [height, width] of the image.
         img = tf.image.resize_image_with_crop_or_pad(img, IMAGE_SIZE, IMAGE_SIZE)
       # Subtract off the mean and divide by the variance of the pixels.
       img = tf.image.per_image_standardization(img)  # 标准化
       return dict(input=img, target=record['label']) 
   	# dict构造字典，{‘input’:img,'target':record['label']}
   ```

   

   

## 网络设计



## 训练模块



## 训练



## 训练保存
