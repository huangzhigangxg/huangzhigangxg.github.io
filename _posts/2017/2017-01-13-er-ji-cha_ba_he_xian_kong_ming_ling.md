---
layout: post

title: 耳机插拔 和 线控命令

date: 2017-01-13 15:32:24.000000000 +09:00

---

## 关于线控：


### IOS7之前：

线控命令属于 远程控制事件，在响应链中传递，需要三个必要条件。
1、UIApplication 开始接受远程控制  [[UIApplication sharedApplication] beginReceivingRemoteControlEvents]
2。接受者 很可能是ApplicationDelegate 或者 ViewController 要可以成为第一响应者 -(BOOL)canBecomeFirstResponder;方法必须返回YES
3；监听线控事件时要保证真正的有音频在播放，并且设置AudioSession
```
[[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:nil];
[[AVAudioSession sharedInstance] setActive:YES error:nil]
```

### IOS7之后：

有MPRemoteCommandCenter可以方便的监听远程控制 不用再有上面的1和2步.


## 关于插拔：


### IOS6之前：
```
[[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:nil];//设置AVAudioSession单例对象的类型

AudioSessionAddPropertyListener(kAudioSessionProperty_AudioRouteChange,audioRouteChangeListenerCallback, (__bridge void *)(self));//调用block函数

//对block函数其中的方法进行实现
void audioRouteChangeListenerCallback (void *inUserData, AudioSessionPropertyID inPropertyID, UInt32 inPropertyValueSize,const void *inPropertyValue ) {
    // ensure that this callback was invoked for a route change
    if (inPropertyID != kAudioSessionProperty_AudioRouteChange) return;
    {
        // Determines the reason for the route change, to ensure that it is not
        //      because of a category change.
        CFDictionaryRef routeChangeDictionary = (CFDictionaryRef)inPropertyValue;
        CFNumberRef routeChangeReasonRef = (CFNumberRef)CFDictionaryGetValue (routeChangeDictionary, CFSTR (kAudioSession_AudioRouteChangeKey_Reason) );
        SInt32 routeChangeReason;
        CFNumberGetValue (routeChangeReasonRef, kCFNumberSInt32Type, &routeChangeReason);
        if (routeChangeReason == kAudioSessionRouteChangeReason_OldDeviceUnavailable) {
            //Handle Headset Unplugged
            NSLog(@"没有耳机！");
        } else if (routeChangeReason == kAudioSessionRouteChangeReason_NewDeviceAvailable) {
            //Handle Headset plugged in
            NSLog(@"有耳机！");
        }
    }
}
```
### IOS6之后：
```
[[AVAudioSession sharedInstance] setActive:YES error:nil];//创建单例对象并且使其设置为活跃状态.

[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(audioRouteChangeListenerCallback:)   name:AVAudioSessionRouteChangeNotification object:nil];//设置通知

//通知方法的实现
- (void)audioRouteChangeListenerCallback:(NSNotification*)notification
{

    NSDictionary *interuptionDict = notification.userInfo;
    NSInteger routeChangeReason = [[interuptionDict valueForKey:AVAudioSessionRouteChangeReasonKey] integerValue];
    switch (routeChangeReason) {
        case AVAudioSessionRouteChangeReasonNewDeviceAvailable:
            NSLog(@"AVAudioSessionRouteChangeReasonNewDeviceAvailable");
            tipWithMessage(@"耳机插入");
            break;
        case AVAudioSessionRouteChangeReasonOldDeviceUnavailable:
            NSLog(@"AVAudioSessionRouteChangeReasonOldDeviceUnavailable");
            tipWithMessage(@"耳机拔出，停止播放操作");
            break;
        case AVAudioSessionRouteChangeReasonCategoryChange:
            // called at start - also when other audio wants to play
            tipWithMessage(@"AVAudioSessionRouteChangeReasonCategoryChange");
            break;
    }
}
```

## 最后代码

```
#pragma mark - headSet
- (void)configureHeadset
{
    [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:nil];
    
    if(![[AVAudioSession sharedInstance] setActive:YES error:nil])
    {
        NSLog(@"Failed to set up a session.");
    }
    
    MPRemoteCommandCenter *remoteCommandCenter = [MPRemoteCommandCenter sharedCommandCenter];
    [remoteCommandCenter.nextTrackCommand setEnabled:YES];
    [remoteCommandCenter.nextTrackCommand addTargetWithHandler:^MPRemoteCommandHandlerStatus(MPRemoteCommandEvent * _Nonnull event) {
        NSLog(@"IOS : nextTrackCommand");
        UnitySendMessage([@"LeVRInputManager" UTF8String], [@"OnButtonDoubleClick" UTF8String], [@"" UTF8String]);
        
        return MPRemoteCommandHandlerStatusSuccess;
    }];
    
    [remoteCommandCenter.togglePlayPauseCommand setEnabled:YES];
    [remoteCommandCenter.togglePlayPauseCommand addTargetWithHandler:^MPRemoteCommandHandlerStatus(MPRemoteCommandEvent * _Nonnull event) {
        NSLog(@"IOS : togglePlayPauseCommand");
        UnitySendMessage([@"LeVRInputManager" UTF8String], [@"OnButtonClick" UTF8String], [@"" UTF8String]);
        return MPRemoteCommandHandlerStatusSuccess;
    }];
    
    [self performSelector:@selector(setHeadSetState) withObject:nil afterDelay:3.0];
    
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(audioRouteChangeListenerCallback:)
                                                 name:AVAudioSessionRouteChangeNotification object:[AVAudioSession sharedInstance]];
}

- (void)audioRouteChangeListenerCallback:(NSNotification*)notification{
    NSDictionary *interuptionDict = notification.userInfo;
    NSInteger routeChangeReason = [[interuptionDict valueForKey:AVAudioSessionRouteChangeReasonKey] integerValue];
    switch (routeChangeReason) {
        case AVAudioSessionRouteChangeReasonNewDeviceAvailable:
            NSLog(@"AVAudioSessionRouteChangeReasonNewDeviceAvailable");
            UnitySendMessage([@"LeVRInputManager" UTF8String], [@"EarphoneState" UTF8String], [@"1" UTF8String]);
            //插入耳机
            break;
        case AVAudioSessionRouteChangeReasonOldDeviceUnavailable:
            NSLog(@"AVAudioSessionRouteChangeReasonOldDeviceUnavailable");
            UnitySendMessage([@"LeVRInputManager" UTF8String], [@"EarphoneState" UTF8String], [@"0" UTF8String]);
            //拔出耳机
            break;
        case AVAudioSessionRouteChangeReasonCategoryChange:
            // called at start - also when other audio wants to play
            NSLog(@"AVAudioSessionRouteChangeReasonCategoryChange");
            break;
    }
}

- (BOOL)isHeadsetPluggedIn {
    AVAudioSessionRouteDescription* route = [[AVAudioSession sharedInstance] currentRoute];
    for (AVAudioSessionPortDescription* desc in [route outputs]) {
        if ([[desc portType] isEqualToString:AVAudioSessionPortHeadphones])
            return YES;
    }
    return NO;
}

- (void)setHeadSetState
{
    if ([self isHeadsetPluggedIn])
    {
        UnitySendMessage([@"LeVRInputManager" UTF8String], [@"EarphoneState" UTF8String], [@"1" UTF8String]);
    }else{
        UnitySendMessage([@"LeVRInputManager" UTF8String], [@"EarphoneState" UTF8String], [@"0" UTF8String]);
    }
}
```