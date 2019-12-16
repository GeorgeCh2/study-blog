### AVVideoExpectedSourceFrameRateKey

原打算使用 AVAssetWriterInput 的 outputSettings 参数中的 AVVideoExpectedSourceFrameRateKey 设置输出帧率，但设置后不起作用，查看文档后发现 ’**This is not used to control the frame rate**‘。遂放弃该参数

![iOS 降低帧率.png](https://upload-images.jianshu.io/upload_images/3911394-fca02a4f1b631120.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




原生参数不起作用的前提下，我们只能考虑其他的方式来达到目的。首先，我们需要理解**帧率**的概念，fps- frame pre sencond（每秒的帧数），那么降低帧率也就是降低每秒包含的帧数，所以我们就可以通过**丢帧**的方式来降低帧率。



### 丢帧

降低帧率其实也就是减少视频中每秒的帧数，那么丢弃掉冗余的视频帧也就达到了我们的目的

![iOS丢帧.png](https://upload-images.jianshu.io/upload_images/3911394-264ca500dbee8791.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### iOS 转码流程

![iOS转码流程.jpg](https://upload-images.jianshu.io/upload_images/3911394-311a781c25100d01.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


目前我们使用的 AVAsset 的转码流程：先将输入视频解码得到一帧帧的视频帧，然后将视频帧送入编码器进行编码得到输出视频。

那么在得到视频帧之后我们就可以选择是否要将该帧送入编码器。

![iOS丢帧流程.png](https://upload-images.jianshu.io/upload_images/3911394-cd076f77156424ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


帧率为 frame count / second, 那么帧间隔就是 1/fps。通过判断两帧之间的时间间隔即判断是否丢弃该帧：

```objective-c
/**
 * 判断是否需要丢弃当前帧
 */
- (CMSampleBufferRef) dropFrame:(CMSampleBufferRef)sample {
    CMSampleBufferRef sampleBufferToWrite = NULL;
    if (sample) {
        if (_videoOutFrameIntervals == 0) {
            // 不需要更改帧率的情况下，直接拷贝当前帧
            CMSampleBufferCreateCopy(nil, sample, &sampleBufferToWrite);
        } else {
            // 是否跳过当前帧
            BOOL drop = true;
            
            // 获取当前帧
            CMTime presentationTime = CMSampleBufferGetPresentationTimeStamp(sample);
            Float64 subPresentationTime = CMTimeGetSeconds(CMTimeSubtract(presentationTime, _previousPstTime));
            if (subPresentationTime == 0 || subPresentationTime >= _videoOutFrameIntervals) {
                // 首帧不跳过;  非首帧的情况下，判断与前一帧的间距是否满足输出帧率（帧间距 >= 1/输出帧率）
                drop = false;
            }
            
            if (!drop) {
                // 不跳过当前帧的情况下
                CMSampleTimingInfo timeInfo = {0};
                CMSampleBufferGetSampleTimingInfo(sample, 0, &timeInfo);
                CMSampleBufferCreateCopyWithNewTiming(nil, sample, 1, &timeInfo, &sampleBufferToWrite);
                
                // 更新前一帧的 presentation 时间
                _previousPstTime = presentationTime;
            }
        }
    }
    
    return sampleBufferToWrite;
}
```

