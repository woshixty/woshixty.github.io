转载自: [什么是usbmuxd?](https://iosre.com/t/%E4%BB%80%E4%B9%88%E6%98%AFusbmuxd-idevice%E9%80%9A%E8%BF%87usb%E4%B8%8E%E6%A1%8C%E9%9D%A2%E7%B3%BB%E7%BB%9F%E9%80%9A%E4%BF%A1%E5%8E%9F%E7%90%86%E5%B0%8F%E7%A7%91%E6%99%AE/1482)

# 什么是usbmuxd? iDevice通过USB与桌面系统通信原理小科普

os x上，苹果有一个服务，叫usbmuxd，这个服务主要用于在USB协议上实现多路TCP连接，将USB通信抽象为TCP通信。苹果的iTunes, XCode，都直接或者间接地用到了这个服务。

那么问题来了，如何让iDevice通过苹果的数据线和mac通信？其实不止是mac，只要pc上提供usbmuxd服务，就可以和iDevice通信，通过TCP.

libimobiledevice，可以在github上找到它。已经将苹果的usbmuxd服务和其他iTunes相关的服务实现跨平台并且开源了。

另外，peerTalk，一个基于usbmuxd服务的开源代码，作为iDevice和OS X通信的一个很好的例子，可以迅速拿来参考，用到自己的app里。既然用到苹果私有（没公开）的协议，那么app能否上架呢？答案是肯定的。并没有用到私有API。某些知名软件比如duet，利用这个usbmuxd上的peerTalk和VNC技术，实现了将iPad作为Mac的显示器的功能，并且在AppStore上架。

另外，树莓派也已经实现了peerTalk。
