
---
layout: post

title: 为什么动画过度效果会突然消失

date: 2016-11-14 15:32:24.000000000 +09:00

---
# 为什么动画过度效果会突然消失

当ViewController再入时 viewDidLoad 中去网路请求数据，有可能导致 [UIView setAnimationsEnabled:NO] 被设置为NO，原因不明，可能因为UIKit 线程不安全。需要在 viewWillAppear 中 重新设置回来 [UIView setAnimationsEnabled:YES];