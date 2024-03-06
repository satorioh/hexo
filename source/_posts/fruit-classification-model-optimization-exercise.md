---
title: 基于CNN的水果分类与模型调优实验
date: 2024-03-06 11:28:19
categories: DL
tags:
  - CNN
  - 模型调优
  - keras
permalink: fruit-classification-model-optimization-exercise
---
记录一次较为满意的实验。调优过程参考了Chollet大神的《Python深度学习》第8章中部分内容

### 一、任务描述
任务：水果图片分类，是一个典型的图像多分类任务

实验环境：colab (tensorflow 2.15.0)

数据集：爬虫从百度图片搜索结果爬取的，包含1036张水果图片，共5个类别（苹果288张、香蕉275张、葡萄216张、橙子276张、梨251张），分类较为均衡
<!--more-->
数据文件夹结构，如下图：
![folder.png](../images/folder.png)

### 二、实验过程
#### 1.数据准备
##### （1）将图片转为dataset
使用keras的`image_dataset_from_directory`接口，可以自动将fruits下的每个子文件夹当作一个class（分类），从而生成dataset(生成的label是文件夹名)

```python
from tensorflow.keras.utils import image_dataset_from_directory

base_dir = 'fruits'
train_ds, validation_ds = image_dataset_from_directory(base_dir, label_mode='categorical',validation_split=0.2,subset="both",batch_size=32,image_size=(180, 180),seed=42)
```
上述代码对dataset做了如下操作：
- validation_split=0.2：划分为训练集（80%）、验证集（20%），
- image_size=(180, 180)：统一图片尺寸为180x180，减小后续训练时的计算量和内存占用
- label_mode='categorical'：label使用何种编码方式。此处为one-hot编码，对应后面使用的损失函数为categorical_crossentropy；如果label_mode设为'int'，则损失函数需要使用sparse_categorical_crossentropy

##### （2）划分测试集
训练集用于模型训练中的参数调整，验证集用于模型超参数确定，测试集则用于最后的模型评估，虽然这里使用的数据集不大，但我还是划分了一个测试集出来
```python
# 计算validation_ds的大小
num_validation_samples = tf.data.experimental.cardinality(validation_ds).numpy() # 9
# 定义我们想要用于验证的样本数量，剩下的将用于测试
num_val_samples = int(num_validation_samples * 0.5) # 4
# 验证集
val_ds = validation_ds.take(num_val_samples)
# 测试集
test_ds = validation_ds.skip(num_val_samples)
```

#### 2.模型构建与训练
##### （1）构建
![img.png](../images/model_net.png)
网络结构如上图，包含3个卷积池化组（用于图像特征提取、降维），并添加了Dropout层（正则化，防止过拟合，因为模型层数多，但相对的数据集并不大），然后是2个全连接层（对特征进行抽象整合，最后输出），构建代码如下：

```python
from keras import layers

inputs = keras.Input(shape=(180, 180, 3))
x = layers.Rescaling(1./255)(inputs) # 归一化
x = layers.Conv2D(filters=32, kernel_size=3, activation="relu")(x) # 使用3x3的卷积核
x = layers.MaxPooling2D(pool_size=2)(x) # 使用window size 2x2、步长2的池化
x = layers.Dropout(0.5)(x) # 50%的随机丢弃率
x = layers.Conv2D(filters=64, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Dropout(0.5)(x)
x = layers.Conv2D(filters=128, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Dropout(0.5)(x)
x = layers.Flatten()(x) # 转成二维格式
x = layers.Dense(512, activation="relu")(x)
x = layers.Dropout(0.5)(x)
outputs = layers.Dense(5, activation="softmax")(x) # 因为有5种类别，所以只需要5个神经元做输出
model = keras.Model(inputs=inputs, outputs=outputs)
```
上述代码在开头还添加了一个Rescaling层，用于归一化（将数据映射到0～1之间，加速模型收敛，提高精度）

##### （2）编译
```python
model.compile(loss="categorical_crossentropy",
              optimizer="rmsprop",
              metrics=["accuracy"])
```
- loss="categorical_crossentropy": 分类交叉熵损失函数
- optimizer="rmsprop"：优化器使用rmsprop（均方根前向梯度下降），一种梯度下降算法的改进版本，可动态调整学习率，提高训练效率
- metrics=["accuracy"]：使用精度（这里是CategoricalAccuracy）作为评估指标

##### （3）训练
```python
history = model.fit(
    train_ds,
    epochs=30,
    validation_data=val_ds)
```
跑30轮，输出结果如下：
```shell
Epoch 1/30
33/33 [==============================] - 9s 93ms/step - loss: 4.8970 - accuracy: 0.2325 - val_loss: 1.5861 - val_accuracy: 0.1641
Epoch 2/30
33/33 [==============================] - 2s 66ms/step - loss: 1.3967 - accuracy: 0.3933 - val_loss: 1.2182 - val_accuracy: 0.5078
Epoch 3/30
33/33 [==============================] - 2s 65ms/step - loss: 0.9339 - accuracy: 0.6651 - val_loss: 1.0185 - val_accuracy: 0.6250
Epoch 4/30
33/33 [==============================] - 2s 59ms/step - loss: 0.7777 - accuracy: 0.7043 - val_loss: 0.8928 - val_accuracy: 0.7422
Epoch 5/30
33/33 [==============================] - 2s 59ms/step - loss: 0.6181 - accuracy: 0.7646 - val_loss: 0.7527 - val_accuracy: 0.7422
Epoch 6/30
33/33 [==============================] - 2s 58ms/step - loss: 0.6319 - accuracy: 0.7876 - val_loss: 0.8239 - val_accuracy: 0.6875
Epoch 7/30
33/33 [==============================] - 2s 58ms/step - loss: 0.4638 - accuracy: 0.8373 - val_loss: 0.5568 - val_accuracy: 0.8359
Epoch 8/30
33/33 [==============================] - 3s 68ms/step - loss: 0.4330 - accuracy: 0.8526 - val_loss: 0.4941 - val_accuracy: 0.8203
Epoch 9/30
33/33 [==============================] - 2s 62ms/step - loss: 0.4230 - accuracy: 0.8632 - val_loss: 0.4629 - val_accuracy: 0.8750
Epoch 10/30
33/33 [==============================] - 2s 58ms/step - loss: 0.2967 - accuracy: 0.9100 - val_loss: 0.8572 - val_accuracy: 0.6719
Epoch 11/30
33/33 [==============================] - 2s 59ms/step - loss: 0.3056 - accuracy: 0.9024 - val_loss: 0.3813 - val_accuracy: 0.8828
Epoch 12/30
33/33 [==============================] - 2s 58ms/step - loss: 0.2053 - accuracy: 0.9254 - val_loss: 0.4152 - val_accuracy: 0.8516
Epoch 13/30
33/33 [==============================] - 2s 68ms/step - loss: 0.2324 - accuracy: 0.9263 - val_loss: 0.3878 - val_accuracy: 0.8750
Epoch 14/30
33/33 [==============================] - 2s 66ms/step - loss: 0.2329 - accuracy: 0.9301 - val_loss: 0.3411 - val_accuracy: 0.9219
Epoch 15/30
33/33 [==============================] - 2s 59ms/step - loss: 0.1377 - accuracy: 0.9665 - val_loss: 0.3957 - val_accuracy: 0.8906
Epoch 16/30
33/33 [==============================] - 2s 59ms/step - loss: 0.1833 - accuracy: 0.9455 - val_loss: 0.2581 - val_accuracy: 0.9609
Epoch 17/30
33/33 [==============================] - 2s 58ms/step - loss: 0.0786 - accuracy: 0.9703 - val_loss: 0.3576 - val_accuracy: 0.8672
Epoch 18/30
33/33 [==============================] - 2s 59ms/step - loss: 0.1816 - accuracy: 0.9598 - val_loss: 0.2799 - val_accuracy: 0.9531
Epoch 19/30
33/33 [==============================] - 2s 70ms/step - loss: 0.0762 - accuracy: 0.9789 - val_loss: 0.2764 - val_accuracy: 0.9453
Epoch 20/30
33/33 [==============================] - 2s 63ms/step - loss: 0.0911 - accuracy: 0.9761 - val_loss: 0.3179 - val_accuracy: 0.9531
Epoch 21/30
33/33 [==============================] - 2s 60ms/step - loss: 0.0966 - accuracy: 0.9818 - val_loss: 0.3839 - val_accuracy: 0.9609
Epoch 22/30
33/33 [==============================] - 2s 59ms/step - loss: 0.0784 - accuracy: 0.9828 - val_loss: 0.2903 - val_accuracy: 0.9531
Epoch 23/30
33/33 [==============================] - 2s 59ms/step - loss: 0.0491 - accuracy: 0.9856 - val_loss: 0.3415 - val_accuracy: 0.9453
Epoch 24/30
33/33 [==============================] - 2s 59ms/step - loss: 0.0317 - accuracy: 0.9914 - val_loss: 0.3351 - val_accuracy: 0.9531
Epoch 25/30
33/33 [==============================] - 2s 70ms/step - loss: 0.0901 - accuracy: 0.9799 - val_loss: 0.3475 - val_accuracy: 0.9531
Epoch 26/30
33/33 [==============================] - 2s 63ms/step - loss: 0.0694 - accuracy: 0.9809 - val_loss: 0.2907 - val_accuracy: 0.9531
Epoch 27/30
33/33 [==============================] - 2s 59ms/step - loss: 0.0029 - accuracy: 1.0000 - val_loss: 0.3122 - val_accuracy: 0.9531
Epoch 28/30
33/33 [==============================] - 2s 59ms/step - loss: 0.1168 - accuracy: 0.9780 - val_loss: 0.3631 - val_accuracy: 0.9531
Epoch 29/30
33/33 [==============================] - 2s 60ms/step - loss: 0.0359 - accuracy: 0.9866 - val_loss: 0.8567 - val_accuracy: 0.8672
Epoch 30/30
33/33 [==============================] - 2s 60ms/step - loss: 0.0501 - accuracy: 0.9885 - val_loss: 0.3891 - val_accuracy: 0.9531

```
#### 3.评估与预测
##### （1）绘制loss和accuracy曲线
```python
import matplotlib.pyplot as plt


def show_history(history):
    loss = history.history['loss']
    val_loss = history.history['val_loss']
    epochs = range(1, len(loss) + 1)
    plt.figure(figsize=(12, 4))
    plt.subplot(1, 2, 1)
    plt.plot(epochs, loss, 'bo', label='Training loss')
    plt.plot(epochs, val_loss, 'b', label='Validation loss')
    plt.title('Training and validation loss')
    plt.xlabel('Epochs')
    plt.ylabel('Loss')
    plt.legend()
    acc = history.history['accuracy']
    val_acc = history.history['val_accuracy']
    plt.subplot(1, 2, 2)
    plt.plot(epochs, acc, 'bo', label='Training acc')
    plt.plot(epochs, val_acc, 'b', label='Validation acc')
    plt.title('Training and validation accuracy')
    plt.xlabel('Epochs')
    plt.ylabel('Accuracy')
    plt.legend()
    plt.show()
    
show_history(history)
```
![loss和accuracy曲线](../images/fruit_history_1.png)
如上图，loss和accuracy都不错，也没有太大的过拟合，说明Dropout还是很有效的

执行一下评估：
```python
test_loss, test_acc = model.evaluate(test_ds)
print(f"Test accuracy: {test_acc:.3f}")
```
结果：0.925
```shell
5/5 [==============================] - 1s 104ms/step - loss: 0.3252 - accuracy: 0.9248
Test accuracy: 0.925
```

##### （2）预测并获取分类报告
```python
from sklearn.metrics import classification_report

# 获取预测结果
y_pred = model.predict(test_ds)

# 获取测试集的真实标签
y_true = np.concatenate([y for x, y in test_ds], axis=0)

y_pred = np.argmax(y_pred, axis=1)
y_true = np.argmax(y_true, axis=1)
```
模型使用softmax输出的是一个二维的概率值（5个分类各自可能的概率大小），而classification_report需要传入一维的真实label和预测值，所以这里使用np.argmax将每个样本中概率最大的值，转为对应的整数索引
```shell
print(y_true)

array([1, 0, 3, 2, 2, 3, 3, 3, 0, 1, 3, 3, 1, 2, 4, 3, 2, 2, 3, 2, 2, 0,
       3, 1, 1, 4, 4, 3, 3, 0, 0, 3, 0, 3, 0, 1, 1, 1, 1, 2, 4, 4, 2, 4,
       1, 0, 0, 4, 4, 1, 1, 0, 2, 0, 3, 4, 3, 1, 2, 1, 3, 1, 3, 0, 3, 0,
       0, 2, 1, 2, 4, 2, 3, 3, 4, 0, 0, 2, 1, 4, 0, 3, 2, 2, 0, 0, 1, 2,
       1, 0, 3, 0, 2, 4, 3, 4, 1, 4, 2, 3, 0, 3, 2, 1, 4, 1, 3, 2, 0, 0,
       1, 4, 0, 3, 4, 1, 3, 1, 2, 0, 3, 4, 0, 1, 4, 1, 0, 3, 4, 4, 4, 3,
       4])
```
最后看下分类报告的结果
```python
# 生成分类报告
report = classification_report(y_true, y_pred)
print(report)
```
```shell
              precision    recall  f1-score   support

           0       0.93      0.96      0.95        28
           1       0.89      0.93      0.91        27
           2       0.95      0.91      0.93        23
           3       0.97      0.97      0.97        31
           4       0.87      0.83      0.85        24

    accuracy                           0.92       133
   macro avg       0.92      0.92      0.92       133
weighted avg       0.92      0.92      0.92       133

```
从support一栏可以看到每个类别实际参与测试的样本数，总体还算均衡；accuracy是0.92，不错的成绩，但还有优化空间
