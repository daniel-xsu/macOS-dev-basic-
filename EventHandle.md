## AppKit 的事件处理

> 在*AppKit*中的事件都处于一个响应的链条中,这个链条是由一个叫做`NSResponder` 的类定义的,这个响应链条其实是一个列表,它里面装满了能够响应用户事件的对象.当用户点击鼠标,或者按下键盘的某个键,或者触摸触控板,都会生成一个`Event`事件,然后在响应链条中寻找可以处理这个事件的对象对事件进行处理.
> 一个对象如果可以处理事件,那么这个对象必须继承自`NSResponder`这个类.在AppKit中,*NSApplication*,*NSWindow*,*NSView*都继承自`NSResponder`

一个NSResponder实例对象有三个组件:事件消息(鼠标,键盘,触控板等产生的),动作消息(action message: 比如NSButton 执行target 的action 方法,就属于一种action消息),和响应链条

一个应用(NSApplication对象)维护着一组窗口(NSWindow)列表,这些窗口都属于这个App,每个窗口对象又维护着一组继承自NSView的对象,这些NSView对象通常用来绘制交互界面以及处理响应事件.  

* 每个应用都拥有一个单利的NSApplication对象来管理主线程的事件循环(main runloop),以及跟踪窗口和菜单的消息,分发事件给相应的对象,建立自动释放池和接收App级别的通知消息.
* NSApplication对象通过run()方法来开启事件循环(event loop).**这个方法在main()函数中**
* 在Xcode项目工程中,NSApplicationMain()类似下面这样的效果:

    ```
    void NSApplicationMain(int argc ,char * argv[]){
         [NSApplication shareApplication];
         [NSBundl loadNibNamed:"main" owner: NSApp];
         [NSApp run];
    }
    ```
* NSApplication 对象通过调用自身的类方法初始化显示的数据环境,然后挂接到macOS系统的`窗口服务`(接收事件)和`显示服务`(显示内容)中.
* NSApplication 的一个重要任务就是从macOS系统的`窗口服务`中接收事件(Event),然后将它们派发到相应的NSResponsder对象.
* NSApplication 会将接收到的Event 转换为NSEvent 对象.
* 所有的鼠标和键盘事件都会被NSApplication 派发到与之关联的某个具体的NSWindow 对象中,但有一种情况例外:**如果按下的是Command(⌘)键,那么所有的NSWindow对象都有机会响应这个事件.**
* NSApplication同时会响应(或派发)接收到的Apple Event(**这个比较重要**),比如应用启动或者被再次打开(reopened),这个最常用的一个使用场景是通过URL打开我们的App(**处理方式与iOS不同哦,需要特别注意呀**),前提是需要使用NSAppleEventManager类对事件进行注册!!,通常都是写在applicationWillFinishedLaunching(_:)这个方法中.

一个窗口对象(NSWindow)处理窗口级别的事件(window-level events)以及将其他事件传递给窗口中的视图对象,同时一个NSWindow还允许通过它的delegate实现自定义窗口的行为方式.

#### 一个事件(Event)是怎样开始传递到应用(Cocoa Application)的?
  *我们这里说的事件,是指用户通过连接到macOS系统中的鼠标,键盘或者触控板,手写笔等硬件设备的具体操作(比如按下鼠标的按键).*
  
  ![Apple event ](https://ws4.sinaimg.cn/large/006tKfTcly1flfe5jo843j30ks0nkwhl.jpg)  
  我们以最常用的`鼠标`或`键盘`操作来说明`事件传递`到应用的过程.当用户按下鼠标或者键盘时:  
  
* 1.硬件设备首先检测到用户的这个操作,然后通过`驱动程序`将这个`操作动作`转换为`操作数据`.
* 2.驱动程序将`操作数据`准备好之后,会调用macOS内核系统的`I/O Kit`,生成一个硬件级别的`事件`.
* 3.驱动程序将这个`事件`发送到macOS系统的`窗口服务`的`事件队列`中.
* 4.驱动程序通知macOS的`窗口服务`,告知其已经添加了一个`事件`到`队列`中待处理.
* 5.macOS的`窗口服务`收到`驱动程序`的消息后,会寻找对应的进程(也就是应用程序).
* 6.当`窗口服务`找到App 进程后,会将`事件`派发到这个应用进程的`runloop`
* 7.当应用进程的`runloop`接收到`事件`后,就开始了`事件`响应机制,从此刻后,将`事件`将遵循`NSResponder`类的处理.
