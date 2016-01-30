title: MengTime实现过程遇到的问题总结
date: 2016-01-07 16:53:54
tags: [Xcode,iOS]

------

### 1. XCTest/XCTest.h file not found
解决方法：
在报错的`target`的`Building Settings`中的`Framwork Search Paths`添加`$(PLATFORM_DIR)/Developer/Library/Frameworks`

### 2. Xcode升级到7.2后，项目中出现"xxx.h" file not found
解决方法：
在项目中的`Header Search Paths`中添加`$(OBJROOT)/UninstalledProducts/$(PLATFORM_NAME/include`
<!--more-->


### 3. App TransportSecurity has blocked a cleartext HTTP (http://) resource load since it isinsecure. Temporary exceptions can be configured via your app's Info.plistfile.
貌似是Xcode7禁用了HTTP明文传输，它不安全，苹果将HTTP协议改成了HTTPS协议。
解决方法：
在`Info.plist`文件中添加`App Transport Security Settings`，`Type`→`Dictionary`，在该项中再添加`Allow Arbitrary Loads`，其`Type`→`Boolean`，`Value`→`YES`

### 4. Error Domain=com.alamofire.error.serialization.response Code=-1016 "Request failed: unacceptable content-type: text/html"
在使用`AFNetWorking 3.0`时，发送网络请求提示如下错误：

    Error Domain=com.alamofire.error.serialization.response Code=-1016 "Request failed: unacceptable content-type: text/html"
在网上查了下，有人说是AFNetworing本身的问题，解析格式不全，所以需要在AFURLResponseSerialization.m中修改一行代码：
``` objectivec
//修改前
self.acceptableContentTypes = [NSSet setWithObjects:@"application/json", @"text/json", @"text/javascript",nil];

//修改后
self.acceptableContentTypes = [NSSet setWithObjects:@"application/json", @"text/json", @"text/javascript", @"text/html",nil];
```
试了一下，确实可以。但这样不是很好，`AFNetworking`在持续更新，不宜修改（在做修改时，`Xcode`会给出提示）,当使用`CocoaPods`更新三方库，则该错误会重复出现。怎样好一点，暂时没去做深入研究，在这个项目中，暂时这样修改的。
### 5. 关闭键盘问题