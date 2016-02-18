## 前言
##### 加强记忆，温故而知新。

## 上交所官网APP
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




