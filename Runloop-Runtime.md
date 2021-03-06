##<center>学习笔记
###1. Runtime
* `runtime` 是一套底层的C语言API，包含很多强大实用的C语言数据类型和C语言函数，平时我们编写的OC代码，底层都是基于 `runtime` 实现的
* `runtime` 有什么作用:   
   能动态产生一个类，一个成员变量，一个方法;能动态修改一个类，一个成员变量，一个方法;能动态删除一个类，一个成员变量，一个方法   
* 常用头文件:   
	`#import <objc/runtime.h> //包含对类、成员变量、属性、方法的操作`
	`#import <objc/message.h> //包含消息机制`  
* 常用方法  
	`class_copyIvarList（）//返回一个指向类的成员变量数组的指针`
	`class_copyPropertyList（）//返回一个指向类的属性数组的指针`  
* `runtime` 在开发中的用途:  
		1. 动态的遍历一个类的所有成员变量，用于字典转模型,归档解档操作。例如：自动解析开源库 `MJExtension` 就是用 `runtime` 实现的  
		2. 交换方法：通过 `runtime` 的方法，可以进行交换方法的实现；一般用自己写的方法（常用在自己写的框架中，添加某些防错措施）来替换系统的方法实现，常用的地方有：在数组中，越界访问程序会崩，可以用自己的方法添加判断防止程序出现崩溃数组或字典中不能添加 `nil` ，如果添加程序会崩，用自己的方法替换系统防止系统崩溃  
		3. 关联对象：有一个问题：“如何给 `NSArray` 添加一个属性（不能使用继承）”，不能用继承，难道用分类 `（Category）` ，但是分类只能添加方法不能添加属性啊，这个时候就要用到 `runtime` 了. 关联对象是指某个OC对象通过一个唯一的key连接到一个类的实例上。例如：`xiaoming` 是 `Person` 类的一个实例，他的 `dog` （一个OC对象）通过一个绳子（key）被他牵着散步，这可以说 `xiaoming` 和 `dog` 是关联起来的，当然 `xiaoming` 可以牵着多个 `dog`   
		4. 关联KVO观察者：有时候我们在分类中使用KVO，推荐使用关联的对象作为观察者，尽量避免对象观察自身   
		 
###2. RunLoop
* `RunLoop` ：其实就是一个 `while` 的死循环。一个线程一次只能执行一个任务，执行完成后线程就会退出。`RunLoop` 可以让线程能随时处理事件并不退出。让线程在没有处理消息时休眠以避免资源占用，在有消息到来时立刻被唤醒。  
* 要让 `RunLoop` 跑起来，必须要给其添加一个有内容的 `mode`, 而且必须要让他 `Run`。 `RunLoop` 跑起来后相当于是一个 `while` 的死循环，后面的代码不会执行。 `RunLoop` 是在程序的入口 `main` 函数中开启  
* `RunLoop` 作用:  
		1.  保持程序持续运行，程序一启动就会开一个主线程，主线程一开起来就会跑一个主线程对应的 `RunLoop` , `RunLoop` 保证主线程不会被销毁，也就保证了程序的持续运行  
		2. 处理APP中的各种事件，比如：触摸事件（手势、`UIScrollView` 的滑动）、定时器事件、`selector` 事件  
		3. 节省CPU资源，提高程序性能。程序运行起来时，当什么操作都没有做的时候，`RunLoop` 会进入休眠状态，当有事情做的时候 `RunLoop` 会被唤起  
*  `RunLoop` 和线程的关系:  
		1. 每条线程都有唯一的一个与之对应的 `RunLoop` 对象  
		2. 主线程的 `RunLoop` 已经自动创建好了，子线程的 `RunLoop` 需要主动创建  
		3. `RunLoop` 在第一次获取时创建，在线程结束时销毁  
		4. 子线程执行完操作之后就会立即释放，即使我们使用强引用引用子线程使子线程不被释放，也不能给子线程添加操作，或者再次开启。如何让子线程永远活着，这时就要用到常驻线程：给子线程开启一个 `RunLoop`  
		5. 主线程的 `RunLoop` 里有两个预置的 `Mode` ： `kCFRunLoopDefaultMode` 和 ` UITrackingRunLoopMode` 。这两个 `Mode` 都已经被标记为 `"Common"` 属性。 `DefaultMode`  是App平时所处的状态， `TrackingRunLoopMode` 是追踪 `ScrollView` 滑动时的状态。   
*  `RunLoop` 的退出：  
		1. 主线程销毁 `RunLoop` 退出  
		2.  `Mode` 中有一些 `Timer`  、`Source` 、`Observer` ，这些保证 `Mode` 不为空时保证 `RunLoop` 没有空转并且是在运行的，当 `Mode` 中为空的时候， `RunLoop` 会立刻退出  
		3. 我们在启动 `RunLoop` 的时候可以设置什么时候停止  
* 自动释放池  
		`RunLoop` 内部有一个自动释放池，当 `RunLoop` 开启时，就会自动创建一个自动释放池，当 `RunLoop` 在休息之前会释放掉自动释放池的东西，然后重新创建一个新的空的自动释放池。只有主线程的 `RunLoop` 会默认启动。也就意味着会自动创建自动释放池，子线程需要在线程调度方法中手动添加自动释放池。  
		
	```
@autorelease {
	// 执行代码
}
``` 
* 代码事例：可以通过下面这段代码进行阻塞某块代码的执行，又不阻塞主线程

	```
while (!done){
	[[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
}
```

	1. `[[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];`  这个是单次获取消息事件的方法，两个参数代表的意思是，获取 `default` 对应的事件消息，以及超时的时间  
	2. 主线程中使用 `while` 死循环，意味着阻塞主线程，此时主线程的 `Runloop` 处于不工作状态；如果 `RunLoop` 不工作，程序将无任何响应（卡页面）；可以通过自定义 `observer` 来观察活动状态  
	3. 既想阻塞某块代码的执行，又不想影响主线程的其他操作，那么就需要手动轮训调用 `RunLoop` 来获取事件消息；保证程序可以正常运行  

### 3. 属性和成员变量
* 在iOS5之前，定义一个属性：成员变量（大括号下的变量）＋ `@property` ＋ `@synthesize` 成员变量三个步骤  
* 在iOS5之后，苹果默认将编译器从GCC转换为LLVM，不需要为属性声明实例变量了。编译器在编译过程中发现没有新的实例变量后，就会生成一个带下划线开头的实例变量。`@property` 声明的属性不仅生成一个带下划线的成员变量，同时也会生成 `setter／getter` 方法。在.m文件中，可以直接实用 `_myString` 实力变量，也可由通过属性 `self.myString` 都是一样的  

	```
	@interface CustomView
{ 
		NSString *name 
}
@end
```
这段代码只是声明一个成员变量，并没有 `setter／getter` 方法。所以访问成员变量时，可以直接访问name，也可以像C＋＋一样用 `self->name` 来访问，但不能用 `self.name` 来访问  
* 属性对成员变量扩充了存取方法  
* 属性可以用一些关键字进行修饰，在必要的地方用不同的关键字进行修饰

### 4. 类方法、实例方法、静态方法
* 类方法：也称静态方法，指的是用 `static` 关键字修饰的方法。此方法属于类本身的方法，不属于类的某一个实例（对象）  
* 当你给一个类写一个方法，如果该方法需要访问某个实例的成员变量时，那么就将该方法定义成实例方法。静态方法正好相反，它不需要访问某个实例的成员变量，它不需要去改变某个实例的状态。我们就把该方法定义成静态方法
	