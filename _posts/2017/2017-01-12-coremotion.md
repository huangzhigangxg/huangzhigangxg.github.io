---
layout: post

title: CoreMotion 检测手机翻转

date: 2017-01-12 15:32:24.000000000 +09:00

---

## 检测设备横屏
今天要实现的需求是 检测设备横屏 采用CoreMotion框架来实现 这个框架里里面封装了“重力加速器”“陀螺仪”“磁力计”三个硬件模块，这三个硬件模块都可以确定，手机旋转的方向和角度。但是每一个设备都有他们的缺点，他们需要相互配合 互相矫正才能使最后的结果准确。

+ 重力加速器：用来测量重力加速度，通过设备姿势的改变，重力的分量作用到X,Y,Z，获得数据得知重力与XYZ轴的夹角。可以想象为一个的篮球，篮球里面有一个铅球，把铅球分别用“上下左右前后”，六只弹簧固定在篮球的中心位置。最下面的一根弹簧受到的作用力就是重力加速度9.8！在框架里的取值为z = -1。 当值 z = 0 时代表 Z轴上没有作用力。fabs(accelerometerData.acceleration.z) < 0.6 代表在Z轴上的力小于重力，说明与重力有夹角，夹角越大则这个值越接近1，夹角越小越接近0.
+ 陀螺仪：用来测量旋转的加速度，通过精密的仪器，算出围绕XYZ轴的加速度的值，通过累计的数据算出，当前手机的朝向，有点是高频测量，缺点就是误差容易积累，到最后误差会很大。
+ 磁力计：磁场中有一个导体，当有电流通过时，磁场会对导体中的电子产生一个横向的洛伦兹力，发生偏转，正负粒子就会聚集在导体的上下两端，产生电压叫霍尔电压，当用三个导体相互垂直。每一个方向上电压值可以看成一个分量。经过计算就能知道设备的朝向。


[这三个仪器的基本原理](http://blog.sina.com.cn/s/blog_7b9d64af0101cu4p.html)

## 最后代码
```
    if (motionManager.accelerometerAvailable)
    {
        [motionManager startAccelerometerUpdatesToQueue:motionManagerQueue withHandler:^(CMAccelerometerData * _Nullable accelerometerData, NSError * _Nullable error) {
            if (fabs(accelerometerData.acceleration.x) > fabs(accelerometerData.acceleration.y) && fabs(accelerometerData.acceleration.z) < 0.6)
            {
                [self setOrientation:DeviceOrientationHorizontal];
            }
            if (fabs(accelerometerData.acceleration.x) < fabs(accelerometerData.acceleration.y) && fabs(accelerometerData.acceleration.z) < 0.6) {
                [self setOrientation:DeviceOrientationVertical];
            }
        }];
    }
```
## 用户加速度

CMDeviceMotionData中的userAcceleration属性，可以知道用户作用到手机上的加速度。从加速度就能知道作用力是多大。比如可以实现这样一个需求，当用户拍击手机左边栏实现一个PopViewController的功能。
```
ClunkViewController * __weak weakSelf = self;
if (manager.deviceMotionAvailable) {
      manager.deviceMotionUpdateInterval = 0.01f;
      [manager startDeviceMotionUpdatesToQueue:[NSOperationQueue mainQueue]
                                         withHandler:^(CMDeviceMotion *data, NSError *error) {
          if (data.userAcceleration.x < -2.5f) {
              [weakSelf.navigationController popViewControllerAnimated:YES];
          }
      }];
  }
  、
  ```

## 用Attitude属性更好的检测设备在空间中角度的变化

在CMDeviceMotionData中有一个attitude属性，是这个类CMAttitude。他有一个方法叫
- (void)multiplyByInverseOfAttitude:(CMAttitude *)attitude;
比方说传入一个设备开始检测时的attitude，就能知道当前的角度和开始的时候，变化多少！

+ [参考](http://www.ithao123.cn/content-1352936.html)



