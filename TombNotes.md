## 前言
##### 加强记忆，温故而知新。
## 目录
- [NSBlockOperation](#NSBlockOperation)
- [FOUNDATION_EXPORT](#FOUNDATION_EXPORT)
- [NSUUID](#NSUUID)
- [NS_ASSUME_NONNULL_BEGIN](#NS_ASSUME_NONNULL_BEGIN)

## 笔记
- #### NSBlockOperation 
##### 相关联的NSOperation、NSBlockOperation、NSInvocationOperation、NSOperationQueue
##### 默认情况下，NSOperation并不具备封装操作的能力，必须使用它的子类，使用NSOperation子类的方式有3种：
a、 自定义子类继承NSOperation，实现内部相应的方法
b、NSBlockOperation
c、 NSInvocationOperation
#####1、 首先介绍如何用NSOperation封装一个操作，后面再结合NSOperationQueue来使用。
自定义NSOperation子类

		#import <Foundation/Foundation.h>
		@protocol NSDefineOprationDelegate <NSObject>
		- (void) handleDelegate;
		@end
		@interface NSDefineOpration : NSOperation
		@property (nonatomic, assign) id <NSDefineOprationDelegate> delegate;
		- (id)initWithDelegate:(id<NSDefineOprationDelegate>) delegate;  
		@end
  实现文件里：
  
		#import "NSDefineOpration.h"
		@implementation NSDefineOpration
		- (id)initWithDelegate:(id<NSDefineOprationDelegate>) delegate
		{
		    if(self = [super init])
		    {
		        self.delegate = delegate;
		    }
		    return self;
		}
		- (void)main
		{  
		    @autoreleasepool {  
		        //do something
		        sleep(15);
		        NSLog(@"op1 handle  on thread num :%@",[NSThread currentThread]);
		        if([self.delegate respondsToSelector:@selector(handleDelegate)])  
		        {  
		            [self.delegate performSelector:@selector(handleDelegate) withObject:nil];  
		        }  
		    }  
		}
		@end
这里的sleep（15）主要用来做一些延时的操作，比如网络下载等。
调用：

		- (void)oprationTest  
		{  
		    NSDefineOpration *op1 = [[NSDefineOpration alloc] initWithDelegate:self];  
		    op1.completionBlock = ^(){  
		        NSLog(@"op1........OK !!");  
		    };  
		    [op1 start];  
		}
注意：因为在实现的main函数里没有使用异步线程处理，导致直接阻塞了主线程，所以使用这种方式一定注意main函数里操作时间过长导致主线程阻塞问题。耗时比较长的都放到其他线程里处理。
##### 2、接下来介绍NSBlockOperation
		NSLog(@"block start");  
		NSBlockOperation *bop2 = [NSBlockOperation blockOperationWithBlock:^{  
		    sleep(15);  
		    NSLog(@"bop2.....handle..... on thread num%@",[NSThread currentThread]);  
		}];  
		[bop2 setCompletionBlock:^{  
		    NSLog(@"bop2........OK !!");  
		}];  
		[bop2 start];
第2行初始化了一个NSBlockOperation对象，它是用一个Block来封装需要执行的操作。
第9行调用了start方法，紧接着会马上执行Block中的内容。
注意：这里还是在当前线程同步执行操作，并没有异步执行，阻塞主线程。
##### 3、接下来介绍NSInvocationOperation
		- (void)invocationOperation  
		{  
		    NSInvocationOperation * op3 = [[NSInvocationOperation alloc] initWithTarget:(id)self selector:@selector(handleInvoOpDelegate) object:nil];  
		    [op3 setCompletionBlock:^{  
		        NSLog(@"op3........OK !!");  
		    }];  
		    [op3 start];  
		}
		
		- (void)handleInvoOpD  
		{  
		    sleep(5);  
		    NSLog(@"op3.....handle.....  on thread num :%@",[NSThread currentThread]);  
		}
NSInvocationOperation比较简单，就是继承了NSOperation，区别就是它是基于一个对象和selector来创建操作，可以直接使用而不需继承来实现自己的操作处理。
##### 4、最后介绍下NSOperationQueue
把NSOperation子类的对象放入NSOperationQueue队列中，该队列就会启动并开始处理它。队列里可以加入很多个NSOperation, 可以把NSOperationQueue看作一个线程池，可往线程池中添加操作（NSOperation）到队列中。线程池中的线程可看作消费者，从队列中取走操作，并执行它。

		[invoOp6 setQueuePriority:NSOperationQueuePriorityHigh];  
		[qu setMaxConcurrentOperationCount:2];  
		[qu addOperation:bkOp3];
参考资料：
[http://blog.csdn.net/crycheng/article/details/21799611](http://blog.csdn.net/crycheng/article/details/21799611 "http://blog.csdn.net/crycheng/article/details/21799611")
- #### FOUNDATION_EXPORT
在项目中遇到这样的定义：

		FOUNDATION_EXPORT NSString *const BaseInfoChannelIDKey;
		FOUNDATION_EXPORT NSString *const BaseInfoUIDKey;
		FOUNDATION_EXPORT NSString *const BaseInfoSIDKey;
		FOUNDATION_EXPORT NSString *const BaseInfoTokenKey;
于是我想起了另外两种种定义：

		extern NSString *const BaseInfoUIDKey;
		#define kMyConstantString @"Hello"
还有另一种定义：

		#define UIKIT_STATIC_INLINE static inline
		UIKIT_STATIC_INLINE UIEdgeInsets UIEdgeInsetsMake(CGFloat top, CGFloat left, CGFloat bottom, CGFloat right) {
		    UIEdgeInsets insets = {top, left, bottom, right};
		    return insets;
		}
##### 1、网上找了一些资料，知道了他们FOUNDATION_EXPORT与extern的区别，这里引用别人的原话：
If you look in NSObjCRuntime.h (in Foundation) you will see that FOUNDATION_EXPORT compiles to extern in C, extern "C" in C++, and other things in Win32. So, it's a bit more compatible. For most projects, this won't make any difference.([链接](http://stackoverflow.com/questions/10953221/foundation-export-vs-extern "链接"))
##### 2、关于FOUNDATION_EXPORT与define(此处并不全面)
使用第一种方法在检测字符串的值是否相等的时候更快.对于第一种你可以直接使用(stringInstance == MyFirstConstant)来比较,而define则使用的是这种.([stringInstance isEqualToString:MyFirstConstant])
哪个效率高,显而易见了.第一种直接比较的是指针地址,而第二个则是一一比较字符串的每一个字符是否相等.
##### 3、对于static inline
它的意思是告诉编译器这个函数是一个静态的内联函数。引入内联函数是为了解决函数调用效率的问题，由于函数之间的调用，会从一个内存地址调到另外一个内存地址，当函数调用完毕之后还会返回原来函数执行的地址。函数调用会有一定的时间开销，引入内联函数就是为了解决这一问题。(没有 call 指令)
消除函数调用产生的开销，适合与小内存函数，频繁执行的函数。

- ####  NSUUID
在2013年3月21日苹果已经通知开发者，从2013年5月1日起，访问UIDIDs的程序将不再被审核通过，替代的方案是开发者应该使用“在iOS 6中介绍的Vendor或Advertising标示符”。
下面我将列出iOS中目前支持的，以及被废弃的唯一标示符方法，并对其做出相应的解释，希望你看了以后针对唯一标示符的使用上，能够做出正确的确定。
##### 1、CFUUID从iOS2.0开始，CFUUID就已经出现了。它是CoreFoundatio包的一部分，因此API属于C语言风格。CFUUIDCreate 方法用来创建CFUUIDRef，并且可以获得一个相应的NSString，如下代码：

		CFUUIDRef cfuuid =CFUUIDCreate(kCFAllocatorDefault); 
		NSString *cfuuidString =(NSString*)CFBridgingRelease(CFUUIDCreateString(kCFAllocatorDefault, cfuuid));
获得的这个CFUUID值系统并没有存储。每次调用CFUUIDCreate，系统都会返回一个新的唯一标示符。如果你希望存储这个标示符，那么需要自己将其存储到NSUserDefaults, Keychain, Pasteboard或其它地方。
示例: 68753A44-4D6F-1226-9C60-0050E4C00067
##### 2、NSUUIDNSUUID在iOS 6中才出现，这跟CFUUID几乎完全一样，只不过它是Objective-C接口。+ (id)UUID 是一个类方法，调用该方法可以获得一个UUID。通过下面的代码可以获得一个UUID字符串：

		NSString *uuid =[[NSUUID UUID] UUIDString];

跟CFUUID一样，这个值系统也不会存储，每次调用的时候都会获得一个新的唯一标示符。如果要存储的话，你需要自己存储。在我读取NSUUID时，注意到获取到的这个值跟CFUUID完全一样（不过也可能不一样）：
示例: 68753A44-4D6F-1226-9C60-0050E4C00067
##### 3、广告标示符（IDFA-identifierForIdentifier）这是iOS 6中另外一个新的方法，advertisingIdentifier 是新框架AdSupport.framework的一部分。ASIdentifierManager单例提供了一个方法advertisingIdentifier，通过调用该方法会返回一个上面提到的NSUUID实例。

		NSString *adId =[[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];

跟CFUUID和NSUUID不一样，广告标示符是由系统存储着的。不过即使这是由系统存储的，但是有几种情况下，会重新生成广告标示符。如果用户完全重置系统（(设置程序 -> 通用 -> 还原 -> 还原位置与隐私) ，这个广告标示符会重新生成。另外如果用户明确的还原广告(设置程序-> 通用 -> 关于本机 -> 广告 -> 还原广告标示符) ，那么广告标示符也会重新生成。关于广告标示符的还原，有一点需要注意：如果程序在后台运行，此时用户“还原广告标示符”，然后再回到程序中，此时获取广告标示符并不会立即获得还原后的标示符。必须要终止程序，然后再重新启动程序，才能获得还原后的广告标示符。之所以会这样，我猜测是由于ASIdentifierManager是一个单例。  

针对广告标示符用户有一个可控的开关“限制广告跟踪”。Nick Arnott的文章中已经指出了。将这个开关打开，实际上什么也没有做，不过这是希望限制你访问广告标示符。这个开关是一个简单的boolean标志，当将广告标示符发到任意的服务器端时，你最好判断一下这个值，然后再做决定。
示例: 1E2DFA89-496A-47FD-9941-DF1FC4E6484A
##### 4、Vendor标示符 (IDFV-identifierForVendor)这种叫法也是在iOS 6中新增的，不过获取这个IDFV的新方法被添加在已有的UIDevice类中。跟advertisingIdentifier一样，该方法返回的是一个NSUUID对象。

		NSString *idfv =[[[UIDevice currentDevice] identifierForVendor] UUIDString];  

苹果官方的文档中对identifierForVendor有如下这样的一段描述 ：    

The value of this property is the same for apps that come from the same vendor running on the same device. A different value is returned for apps on the same device that come from different vendors, and for apps on different devices regardless of vendor.   

如果满足这样的条件，那么获取到的这个属性值就不会变：相同的一个程序里面-相同的vindor-相同的设备。如果是这样的情况，那么这个值是不会相同的：相同的程序-相同的设备-不同的vindor，或者是相同的程序-不同的设备-无论是否相同的vindor。  

看完上面的内容，我有这样的一个疑问“vendor是什么”。我首先想到的是苹果开发者账号。但事实证明这是错误的。接着我想可能是有一个AppIdentifierPrefix东西，跟钥匙串访问一样，可以在多个程序间共享。同样，这个想法也是的。最后证明，vendor非常简单：一个Vendor是CFBundleIdentifier（反转DNS格式）的前两部分。例如，com.doubleencore.app1 和 com.doubleencore.app2  得到的identifierForVendor是相同的，因为它们的CFBundleIdentifier 前两部分是相同的。不过这样获得的identifierForVendor则完全不同：com.massivelyoverrated 或 net.doubleencore。  

在这里，还需要注意的一点就是：如果用户卸载了同一个vendor对应的所有程序，然后在重新安装同一个vendor提供的程序，此时identifierForVendor会被重置。
	
	示例: 599F9C00-92DC-4B5C-9464-7971F01F8370
##### 5、UDID在之前的版本中是可用的，但是在iOS5以及之后的版本中，以及被弃用了。虽然，这个UDID用得很广泛，但是，不得不说的是，它在慢慢的远离开发者，不能在考虑使用UDID了。至于这个标示符是转为私有方法，或者完全从以后的iOS版本中移除，还有待观察。不过，这个UDID在部署企业级签名程序时，非常方便。获取UDID的方法如下：  

		NSString *udid =[[UIDevice currentDevice] uniqueIdentifier];  
示例: bb4d786633053a0b9c0da20d54ea7e38e8776da4
##### 6、OpenUDID在iOS 5发布时，uniqueIdentifier被弃用了，这引起了广大开发者需要寻找一个可以替代UDID，并且不受苹果控制的方案。由此OpenUDID成为了当时使用最广泛的开源UDID替代方案。OpenUDID在工程中实现起来非常简单，并且还支持一系列的广告提供商。

		NSString *openUDID = [OpenUDID value];
OpenUDID利用了一个非常巧妙的方法在不同程序间存储标示符 — 在粘贴板中用了一个特殊的名称来存储标示符。通过这种方法，别的程序（同样使用了OpenUDID）知道去什么地方获取已经生成的标示符（而不用再生成一个新的）。    

之前已经提到过，在将来，苹果将开始强制使用advertisingIdentifier 或identifierForVendor。  如果这一天到来的话，即使OpenUDID看起来是非常不错的选择，但是你可能不得不过渡到苹果推出的方法。    

	示例: 0d943976b24c85900c764dd9f75ce054dc5986ff

- ####  NS_ASSUME_NONNULL_BEGIN 

	我们都知道在swift中，可以使用!和?来表示一个对象是optional的还是non-optional，如view?和view!。而在Objective-C中则没有这一区分，view即可表示这个对象是optional，也可表示是non-optioanl。这样就会造成一个问题：在Swift与Objective-C混编时，Swift编译器并不知道一个Objective-C对象到底是optional还是non-optional，因此这种情况下编译器会隐式地将Objective-C的对象当成是non-optional。
	
	为了解决这个问题，苹果在Xcode 6.3引入了一个Objective-C的新特性：nullability annotations。这一新特性的核心是两个新的类型注释：__nullable和__nonnull。从字面上我们可以猜到，__nullable表示对象可以是NULL或nil，而__nonnull表示对象不应该为空。
	
	我们来看看以下的实例：  
	
		@interface TestNullabilityClass () 
		@property (nonatomic, copy) NSArray * items; 
		- (id)itemWithName:(NSString * __nonnull)name; 
		@end 
		@implementation TestNullabilityClass 
		... 
		- (void)testNullability { 
		[self itemWithName:nil]; // 编译器警告：Null passed to a callee that requires a non-null argument 
		} 
		- (id)itemWithName:(NSString * __nonnull)name { 
		return nil; 
		} 
		@end 
	
	不过这只是一个警告，程序还是能编译通过并运行。
	
	事实上，在任何可以使用const关键字的地方都可以使用__nullable和__nonnull，不过这两个关键字仅限于使用在指针类型上。有以下三种方式：  

		- (nullable id)itemWithName:(NSString * nonnull)name 
		@property (nonatomic, copy, nonnull) NSArray * items;
		@property (nonatomic, copy) NSArray * __nonnull items;  

	如果需要每个属性或每个方法都去指定nonnull和nullable，是一件非常繁琐的事。苹果为了减轻我们的工作量，专门提供了两个宏：NS_ASSUME_NONNULL_BEGIN和NS_ASSUME_NONNULL_END  
	
		NS_ASSUME_NONNULL_BEGIN
		@interface TestNullabilityClass ()  
		@property (nonatomic, copy) NSArray * items; 
		- (id)itemWithName:(nullable NSString *)name; 
		@end 
		NS_ASSUME_NONNULL_END


##  ios沙盒机制
1. 每个应用程序都在自己的沙盒内
1. 不能随意跨越自己的沙盒去访问别的应用程序沙盒的内容
1. 应用程序向外请求或接收数据都需要经过权限认证  

```objective-c
//获取根目录，前往homePath可以看到该文件夹下有Documents、Library、tmp三个文件夹。
    NSString *homePath = NSHomeDirectory();
```

#### Documents 
苹果建议将程序中创建的或在程序中浏览到的文件数据保存在该目录下，iTunes备份和恢复的时候会包括此目录；  
```objective-c
//获取Documents文件夹目录,第一个参数是说明获取Doucments文件夹目录，第二个参数说明是在当前应用沙盒中获取，所有应用沙盒目录组成一个数组结构的数据存放
    NSArray *docPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsPath = [docPath objectAtIndex:0];
```

#### Library 
存储程序的默认设置或其它 状态信息；Library/Caches：存放缓存文件，iTunes不会备份此目录，此目录下文件不会在应用退出删除

```objective-c
//获取Cache目录
    NSArray *cacPath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    NSString *cachePath = [cacPath objectAtIndex:0];
```
#### tmp
创建和存放临时文件的地方。

```objective-c
   NSString *tempPath = NSTemporaryDirectory();
```
## UIAppearance 的使用
在iOS 5以前，自定义原生控件的外观并没有原生支持，因此开发人员感觉很麻烦。开发人员经常面临的问题是修改一个控件所有实例的外观。解决这个问题的正确方法是重写一遍控件。但由于这么做非常费时，一些开发人员开始覆盖或混写一些方法，如drawRect:  

从iOS 5开始，苹果通过两个协议（UIAppearance和UIAppearanceContainer）规范了对许多UIKit控件定制的支持。所有遵循UIAppearance协议的UI控件通过定制都可以呈现各种外观。不仅如此，UIAppearance协议甚至允许开发者基于控件所属的区域指定不同的外观。也就是说，当某个控件包含在特定视图中时，可以指定它的外观（如UIBarButtonItem的tintColor）。也可以获取该控件类的外观代理对象，用该代理定制外观来实现，下面来看一个例子。  

要定制应用中所有条形按钮的颜色，可以在UIBarButtonItem的外观代理中设置tintColor：
```
[[UIBarButtonItem  appearance]  setTintColor:[UIColor  redColor]];
```
注意，iOS 4的时候setTintColor方法就在UIBarButtonItem中了，但它只会作用到某个特定的控件实例，而不是所有的此类控件。借助外观代理对象，我们可以定制使用上述类创建的任意对象的外观。  

同样，可以根据内部包含的视图采用如下方法来定制控件的外观：
```
[[UIBarButtonItem appearanceWhenContainedIn:[UINavigationBar class], nil] setTintColor:[UIColor redColor]];
```
第一个参数是以nil结尾的所有容器类的列表，包括UINavigatorBar、UIPopOverController等遵循UIAppearanceContainer协议的类。  

#### UI_APPEARANCE_SELECTOR 
从iOS 5开始，大多数UI元素都增加了对UIAppearance协议的支持。此外，iOS 5中类似于UISwitch的控件允许我们方便地将on开关的颜色变成设计师选定的颜色。现在，怎么确定哪些情况下能够通过UIKit的外观代理来定制所有元素（以及元素中的哪些属性）呢？有两种方式。老办法是查阅文档，另一个办法是大多数开发人员使用的快捷方式：读头文件。打开对应的UIKit元素的头文件，其中所有带有UI_APPEARANCE_SELECTOR标记的属性都支持通过外观代理来定制。举个例子，UINavigationBar.h中的tintColor属性带有UI_APPEARANCE_SELECTOR标记：
```
@property(nonatomic,retain) UIColor      *tintColor    UI_APPEARANCE_SELECTOR;
```
意味着可以调用
```
[[UINavigationBar   appearance]  setTintColor:newColor];
```  

我们也可以自定义支持appearance的方法或属性，例：
```objective-c
@interface Button:UIButton<UIAppearance>
@property(nonatomic,strong)UIColor *bColor UI_APPEARANCE_SELECTOR;
@end;
@implementation Button
-(void)setBColor:(UIColor *)bColor
{
    _bColor = bColor;
    self.backgroundColor = _bColor;
}
```
这样使用：
```
[Button appearance].bColor = [UIColor greenColor];
```
##### 请注意＊使用appearance设置UI效果最好采用全局的设置，在所有界面初始化前开始设置，否则可能失效。

