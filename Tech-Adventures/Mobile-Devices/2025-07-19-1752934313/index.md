转载自: [使用usbmuxd服务，通过USB连接与PC端、Mac端实现通信，Peertalk的使用](https://www.jianshu.com/p/eba133891ec6)

# 使用usbmuxd服务，通过USB连接与PC端、Mac端实现通信，Peertalk的使用

## 一、usbmuxd 介绍

usbmuxd 是苹果的一个服务，这个服务主要用于在USB协议上实现多路TCP连接，将USB通信抽象为TCP通信。苹果的iTunes、Xcode，都直接或间接地用到了这个服务。

iTunes使用 usbmux 与 iphone 通信, 它提供了一个USB - TCP的转换服务， 这个服务在Mac端是由/System/Library/PrivateFrameworks/MobileDevice.framework/Resources/usbmuxd提供的, 当然, 开机自动启动。

它创建了一个Unix Domain Socket 在 /var/run/usbmuxd. usbmuxd服务程序监控iPhone在USB口上的连接, 当它监控到iPhone以用户模式连接到USB, (相对的是recovery模式), usbmuxd服务程序就会连接到这个/var/run/usbmuxd的TCP端口, 并开始成为一个USB - TCP 请求转发器

那么,如果想编写个第三方程序与iphone进行通信,实现类似iTunes的功能, 你的程序可以通过usbmuxd! 建立一个TCP连接到/var/run/usbmuxd端口, 根据协议发送对应的请求包, usbmuxd服务会将请求转发到USB的iPhone上。

[peertalk](https://link.jianshu.com/?t=https://github.com/rsms/peertalk)，一个基于usbmuxd服务的开源代码，可以实现 iPhone 与 Mac 通信。

[libimobiledevice](https://link.jianshu.com/?t=https://github.com/Polyfun/libimobiledevice-windows)，在可以PC端提供usbmuxd服务，实现 iPhone 与 windows 通信。

## 二、Peertalk 的使用：iPhone 与 Mac 通信

### iOS 端

1、创建 channel，监听指定端口

```objectivec
// 创建 channel
PTChannel *channel = [PTChannel channelWithDelegate:self];
// 监听指定端口,PTExampleProtocolIPv4PortNumber自定义端口号
[channel listenOnPort:PTExampleProtocolIPv4PortNumber IPv4Address:INADDR_LOOPBACK callback:^(NSError *error) {
    if (error) { // 创建监听失败
    
    } else { // 创建监听成功 
    
    }
}];
```

2、实现 Channel 的代理方法

```objectivec
@protocol PTChannelDelegate <NSObject>

@required
// 收到信息
- (void)ioFrameChannel:(PTChannel*)channel didReceiveFrameOfType:(uint32_t)type tag:(uint32_t)tag payload:(PTData*)payload;

@optional
// 收到消息调用，如回复NO，则忽略这条消息
- (BOOL)ioFrameChannel:(PTChannel*)channel shouldAcceptFrameOfType:(uint32_t)type tag:(uint32_t)tag payloadSize:(uint32_t)payloadSize;

// 出错回调
- (void)ioFrameChannel:(PTChannel*)channel didEndWithError:(NSError*)error;

// 连接成功回调
- (void)ioFrameChannel:(PTChannel*)channel didAcceptConnection:(PTChannel*)otherChannel fromAddress:(PTAddress*)address;

@end
```

3、连接成功后，会发送设备信息

### Mac 端

1、监听USB设备的连接/断开

```objectivec
// 开始监听设备的连接与断开
- (void)startListeningForDevices {
    NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
    
    // 监听设备连接
    [nc addObserverForName:PTUSBDeviceDidAttachNotification object:PTUSBHub.sharedHub queue:nil usingBlock:^(NSNotification *note) {
        NSNumber *deviceID = [note.userInfo objectForKey:@"DeviceID"];
        dispatch_async(notConnectedQueue_, ^{
            if (!connectingToDeviceID_ || ![deviceID isEqualToNumber:connectingToDeviceID_]) {
                // 断开现有连接
                [self disconnectFromCurrentChannel];
                connectingToDeviceID_ = deviceID;
                connectedDeviceProperties_ = [note.userInfo objectForKey:@"Properties"];
                // 重新连接
                [self enqueueConnectToUSBDevice];
            }
        });
    }];
    
    // 监听设备断开
    [nc addObserverForName:PTUSBDeviceDidDetachNotification object:PTUSBHub.sharedHub queue:nil usingBlock:^(NSNotification *note) {
        NSNumber *deviceID = [note.userInfo objectForKey:@"DeviceID"];
        if ([connectingToDeviceID_ isEqualToNumber:deviceID]) {
            connectedDeviceProperties_ = nil;
            connectingToDeviceID_ = nil;
            if (connectedChannel_) { // 关闭连接通道
                [connectedChannel_ close];
            }
        }
    }];
}
```

2、连接设备

```objectivec
// 连接设备
- (void)connectToLocalIPv4Port {
    // 创建通道，设置代理
    PTChannel *channel = [PTChannel channelWithDelegate:self];
    channel.userInfo = [NSString stringWithFormat:@"127.0.0.1:%d", PTExampleProtocolIPv4PortNumber];
    // 连接指定端口地址，与iOS端设置保持一致
    [channel connectToPort:PTExampleProtocolIPv4PortNumber IPv4Address:INADDR_LOOPBACK callback:^(NSError *error, PTAddress *address) {
        if (error) { // 连接失败
            
        } else { // 连接成功
            
        }
    }];
}
```

3、连接成功会，会收发送 ping、pong 心跳数据

## 三、libimobiledevice、Peertalk 的使用：iPhone 与 windows 通信

### 1、实现原理

windows端 通过 libimobiledevice 运行 usbmuxd 的多路复用守护进程，该进程的作用是建立本地端口和远程端口的转发，实现usb到tcp的转换服务

### 2、安装服务

windows端首先要安装苹果公司提供的相关服务，才能实现通信功能。服务名称为：AppleApplicationSupport和AppleMobileDeviceSupport

### 3、规定协议

首先指定 ip地址和端口，端口号建议大些，以免与苹果系统应用端口重复。如：127.0.0.1:62345。PC端可以通过端口转发实现。  
定义相同结构体数据，以便数据的加密、解析。PC端可和Peertalk定义的协议保持一致。如：

```cpp
// 数据头结构体
typedef struct _PTFrame {
  uint32_t version;     // 对应版本
  uint32_t type;        // 数据类型
  uint32_t tag;         // tag标记
  uint32_t payloadSize; // 数据大小
} PTFrame;

// 数据类型
enum {
  PTExampleFrameTypeDeviceInfo = 100,  // 设备信息
  PTExampleFrameTypeTextMessage = 101, // 文本数据
  PTExampleFrameTypePing = 102,        // Ping
  PTExampleFrameTypePong = 103,        // Pong
};
```

### 4、联调测试

定好协议后，指定相同ip地址和端口，进行测试。如连接失败，可尝试更换端口号重试。先调试连接，然后再收发数据，最后进行数据处理。  
  
  
参考链接1:[http://blog.csdn.net/u010343361/article/details/50539401](https://link.jianshu.com/?t=http://blog.csdn.net/u010343361/article/details/50539401)  
参考链接2:[https://github.com/Polyfun/libimobiledevice-windows](https://link.jianshu.com/?t=https://github.com/Polyfun/libimobiledevice-windows)  
参考链接3:[https://github.com/rsms/peertalk](https://link.jianshu.com/?t=https://github.com/rsms/peertalk)
