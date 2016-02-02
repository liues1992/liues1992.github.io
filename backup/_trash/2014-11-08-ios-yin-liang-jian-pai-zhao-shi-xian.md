---
layout: post
title: "iOS音量键拍照实现"
description: ""
category: "default"
tags: "[]"
---
要解决几个问题
---
1. 能够监听音量键按下事件
2. 不改变原来的音量
3. 拍照时音量指示层不能出现

###监听音量键按下事件
第1个问题的很容易搜索到：解决方法是使用notification

```smalltalk
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(volumeChanged:)
                                             name:@"AVSystemController_SystemVolumeDidChangeNotification"
                                           object:nil];

- (void)volumeChanged:(NSNotification *)note {
	if ([UIApplication sharedApplication].applicationState == UIApplicationStateActive && 
	[note.userInfo[@"AVSystemController_AudioVolumeChangeReasonNotificationParameter"] isEqualToString:@"ExplicitVolumeChange"]) {
		//take photo if possile
	}
    
}
```
其中的`AVSystemController_SystemVolumeDidChangeNotification`官方文档中并没有任何说明，有可能被官方当做Private API，从而拒绝上架App Store。但目前App Store确实有其他上架的可以按音量键拍照的应用。

这里有几个要注意的问题是：
- 似乎当前 `AVAudioSession` 为Active时才能接受到这个通知。这里我没有详细去验证，在我自己的应用里在应用becomeActive时

```smalltalk
AVAudioSession *session = [AVAudioSession sharedInstance];
[session setCategory:AVAudioSessionCategoryPlayback
              withOptions:AVAudioSessionCategoryOptionMixWithOthers
                    error:nil];
[session setActive:YES error:nil];

```
- 不止是音量改变时会触发这个通知，所以需要检查
    [note.userInfo[@"AVSystemController_AudioVolumeChangeReasonNotificationParameter"] isEqualToString:@"ExplicitVolumeChange"]
- 应用处于后台时也会收到这个通知 所以需要判断 
    [UIApplication sharedApplication].applicationState == UIApplicationStateActive

###不改变原来的音量
如果做到了上一步，那么按下音量键拍照应该可以实现了，但是这样系统的音量也变化了。如果你在播放音乐，音乐声会变化。这不是我们想要的。

但是没有办法去阻止系统音量的改变，所以唯一的办法是系统音量变了之后再改回来。做到这一点的思路是：1.保存系统原来的音量 2.拍照时把音量改回之前的音量。

获取音量的API是  `[[AVAudioSession sharedInstance] outputVolume];`
并且确保**当前的AudioSession是Active状态** (在第一步我已经做了`[[AVAudioSession sharedInstance] setActive:YES error:nil]`)

修改音量则要麻烦一些，系统并不提供改变音量的API，但是有一个`MPVolumeView`可以提供界面给用户用来调节音量。通过这个东西可以迂回实现调节音量的功能。
这里的hack是提供一个看不见的MPVolumeView，得到其中UISlider，通过改变slider的value来改变系统音量。严格意义上说也是属于Private API，存在被Apple拒绝可能性。

```smalltalk
    CGRect frame = CGRectMake(-1000, -1000, 100, 100);
    MPVolumeView *volumeView = [[MPVolumeView alloc] initWithFrame:frame];
    [volumeView sizeToFit];
    self.volumeView = volumeView;
    UISlider *volumeSlider = nil;
    [[volumeView subviews] enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
        if ([obj isKindOfClass:[UISlider class]]) {
            volumeSlider = obj;
            *stop = YES;
        }
    }];
    .....
    slider.value = someVolume
```
>**注意**:slider.value = someVolume由于会改变音量，所以也会触发上面的notification。应当在适当时机调用。

###拍照时音量指示层不能出现
做到了第2点其实这一点就很容易了。只要吧volumeView添加为任意一个view的subview，系统会认为我已经有个MPVolumeView来指示音量了，不需要那个浮层了。不在拍照界面是应该把volumeView移除，恢复正常音量调节指示功能。

###Done













