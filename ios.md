## Basic Setup

NexPlayer synchronization can be enabled by setting the following property:


```swift
player.setProperty(NXPropertyEnableSpdSyncToGlobalTime, value: 1 as NSObject)
```

To set the SPD value from the client side you can set the SPD value with the property ```NXPropertySuggestedPresentationDelayTime``` as shown below:

```swift
player.setProperty(NXPropertySuggestedPresentationDelayTime, value: 5000 as NSObject)
```

Note that if the value of ```NXPropertySuggestedPresentationDelayTime``` is too large, the player may not find the delayed segment provided by the live content server. You should optimise the presentation delay according to your use case.


## Advanced configuration

You can control the synchronization behaviour further by adjusting the below
properties.

### Synchronization Accuracy

By default, our synchronization feature depends on the server time, which is only in seconds. When this option is turned on, device time will be used instead. This property makes the synchronization more accurate as it is in milliseconds but keep in mind that if the device time is not set correctly, there might be discrepancies between different devices.

```swift
player.setProperty(NXPropertyEnableSpdSyncToDeviceTime, value: 1 as NSObject)
```

Default: 0

Values:

- 0: Disabled
- 1: Enabled


### Synchronization Speed Control Range

Player will keep the syncronization by controlling the playback speed as long as the playback position is within this range. You can adjust this threshold with the ```NXPropertySpdSyncDiffTime``` property.

```swift
player.setProperty(NXPropertySpdSyncDiffTime, value: 300 as NSObject)
```

Unit: msec (1/1000 sec)

Default: 300 (300 msec)

### Synchronization Seek Range

If playback is out of synchronization more than this value, the player will make a seek to synchronize the video rather than changing the playback speed.

```swift
player.setProperty(NXPropertySpdTooMuchDiffTime, value: 5000 as NSObject)
```

Unit: msec (1/1000 sec)

Default: 5000 (5 seconds)


## Requirements

- Our NexPlayer iOS SDK supports iOS 11 and above but NexPlayer Multiview feature requires an high-end device as it depends on the device performance. At least iOS 13+ should be targeted and device performance should be considered.

- You should make sure there is enough distance from the live edge to provide
a smooth playback which should be adjusted with *suggestedPresentationDelay* and ```NXPropertySuggestedPresentationDelayTime``` properties. If there is not enough space to buffer from the live edge, playback might be effected.

- Video encoders should be syncronised and should be embedding correct timestamp for the  video streams

- For HLS, you should set ```NXPropertySuggestedPresentationDelayTime``` property as mentioned
above.