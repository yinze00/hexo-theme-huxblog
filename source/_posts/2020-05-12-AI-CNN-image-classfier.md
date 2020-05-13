---
title: AI-CNN-Image-Classfier
date: 2020-05-13 17:29:53
tags: AI
header-img: "AI.jpg"
author: "yinze"
---
>

<font face="黑体" color=black size=6>AI CNN-Image-Classfier Report</font>





#### 1. 选型

​	卷积神经网络（Convolutional Neural Network, CNN）是一种前馈神经网络，它的人工神经元可以响应一部分覆盖范围内的周围单元，对于大型图像处理有出色表现。

#### 2. 运行环境

* 选择离线训练，用的实验室服务器双路1080ti
* tensorflow-gpu 1.15

 * keras 2.3.1
* Ubuntu 14.04
* cuda v10.2  GTX 1080ti 
* python 3.6.7
* VScode 

#### 3. 具体实现

 * train.py

   ```python
   # load image from directory use image dataGenerator
   datagen = ImageDataGenerator(
       rescale=1. / 255,
       shear_range=0.1,
       zoom_range=0.1,
       horizontal_flip=True
   )
   train_image_data = datagen.flow_from_directory(
       'datasets/la1ji1fe1nle4ishu4ju4ji22-momodel/dataset-resized',
       target_size=(150, 150),
       batch_size=32,
       class_mode='categorical'
   )
   print(train_image_data)
   #  create model to train our image dataset cnn model
   
   model = Sequential()
   
   model.add(Conv2D(32, (3, 3), input_shape=(150, 150, 3)))
   model.add(Activation('relu'))
   model.add(MaxPooling2D(pool_size=(2, 2)))
   
   model.add(Conv2D(32, (3, 3)))
   model.add(Activation('relu'))
   model.add(MaxPooling2D(pool_size=(2, 2)))
   
   model.add(Conv2D(64, (3, 3)))
   model.add(Activation('relu'))
   model.add(MaxPooling2D(pool_size=(2, 2)))
   
   model.add(Flatten())    
   model.add(Dense(64))
   model.add(Activation('relu'))
   
   model.add(Dropout(0.5))
   model.add(Dense((6)))
   model.add(Activation('softmax'))
   
   print(model.summary())
   print(model.output_shape)
   
   #  compile model categorical_crossentropy
   model.compile(
       loss='categorical_crossentropy',
       optimizer='adam',
       metrics=['accuracy']
   )
   # fit the model
   model.fit_generator(
       train_image_data,
       epochs=500,
       steps_per_epoch=2307 // 32
   )
   # save the model
   model.save('results/final.h5')
   ```

 * test.py

   ```python
   from keras.models import load_model
   from keras.preprocessing import image
   import numpy as np
   import os
   import PIL.Image
   
   # -------------------------- 请加载您最满意的模型 ---------------------------
   # 加载模型(请加载你认为的最佳模型)
   # 加载模型,加载请注意 model_path 是相对路径, 与当前文件同级。
   # 如果你的模型是在 results 文件夹下的 dnn.h5 模型，则 model_path = 'results/dnn.h5'
   model_path = 'results/final.h5'
   
   # 加载模型，如果采用keras框架训练模型，则 model=load_model(model_path)
   model = load_model(model_path)
       
   # ---------------------------------------------------------------------------
   
   def predict(img):
       """
       加载模型和模型预测
       主要步骤:
           1.图片处理
           2.用加载的模型预测图片的类别
       :param img: PIL.Image 对象
       :return: string, 模型识别图片的类别, 
               共 'cardboard','glass','metal','paper','plastic','trash' 6 个类别
       """
       # -------------------------- 实现模型预测部分的代码 ---------------------------
       # 获取图片的类别，共 'cardboard','glass','metal','paper','plastic','trash' 6 个类别
       # 把图片转换成为numpy数组
       
       img = img.resize((150,150,3),PIL.ANTLIAS)
   
       img = image.img_to_array(img)
       img_array = img/255
       img_array = np.expand_dims(img_array, axis=0)
   
       result = model.predict(img_array)
       result = np.argmax(result)
       labels = {0: 'cardboard', 1: 'glass', 2: 'metal', 3: 'paper', 4: 'plastic', 5: 'trash'}
   
       # 获取输入图片的类别
       y_predict = labels[result]
   
       # -------------------------------------------------------------------------
       
       # 返回图片的类别
       return y_predict
   ```

   

#### 4. 效果

##### 	4.1 调参

* 5-CNN

| Epoch |  Loss  | Acurracy |
| :---: | :----: | :------: |
|  20   |  1.41  |   0.42   |
|  50   |  1.08  |   0.56   |
|  100  |  0.62  |   0.76   |
|  200  |  0.48  |   0.83   |
|  400  |  0.24  |   0.89   |
|  500  | 0.1125 |  0.9651  |

##### 	4.2 训练	![](C:\Users\Yinze\Desktop\1234.JPG)

##### 	4.3 实测效果

* 随机挑选16张图片并预测和显示

![](C:\Users\Yinze\Desktop\123.png)



#### 5. 感想

* 炼丹真是玄之又玄
* 服务器跑了一晚上果然接近0.99了
* 忘了参加那个AI对抗比赛，但是看了下目前榜一的accuracy(0.91) 比我的模型0.9651还弱，后悔了！！！

