从图中看，NSInputStream可以从文件、socket和NSData对象中获取数据；NSOutputStream可以将数据写入文件、socket、内存缓存和NSData对象中。这三处类主要处理一些比较底层的任务。

流对象有一些相关的属性。大部分属性是用于处理网络安全和配置的，这些属性统称为SSL和SOCKS代理信息。两个比较重要的属性是：


NSStreamDataWrittenToMemoryStreamKey：允许输出流查询写入到内存的数据
NSStreamFileCurrentOffsetKey：允许操作基于文件的流的读写位置


可以给流对象指定一个代理对象。如果没有指定，则流对象作为自己的代理。流对象调用唯一的代理方法stream:handleEvent:来处理流相关的事件：


对于输入流来说，是有可用的数据可读取事件。我们可以使用read:maxLength:方法从流中获取数据
对于输出流来说，是准备好写入的数据事件。我们可以使用write:maxLength:方法将数据写入流


Cocoa中的流对象与Core Foundation中的流对象是对应的。我们可以通过toll-free桥接方法来进行相互转换。NSStream、NSInputStream和NSOutputStream分别对应CFStream、CFReadStream和CFWriteStream。但这两者间不是完全一样的。Core Foundation一般使用回调函数来处理数据。另外我们可以子类化NSStream、NSInputStream和NSOutputStream，来自定义一些属性和行为，而Core Foundation中的流对象则无法进行扩展。

上面主要介绍了iOS中流的一些基本概念，我们下面将介绍流的具体使用，首先看看如何从流中读取数据。

从输入流中读取数据

从一个NSInputStream流中读取数据主要包括以下几个步骤：


从数据源中创建和初始化一个NSInputStream实例
将流对象放入一个run loop中并打开流
处理流对象发送到其代理的事件
当没有更多数据可读取时，关闭并销毁流对象。
准备流对象


要使用一个NSInputStream，必须要有数据源。数据源可以是文件、NSData对象和网络socket。创建好后，我们设置其代理对象，并将其放入到run loop中，然后打开流。代码清单1展示了这个准备过程.

代理清单1

 
复制代码
- (void)setUpStreamForFile:(NSString *)path
{
    NSInputStream *inputStream = [[NSInputStream alloc] initWithFileAtPath:path];
    inputStream.delegate = self;
    [inputStream scheduleInRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];
    [inputStream open];
}


在流对象放入run loop且有流事件(有可读数据)发生时，流对象会向代理对象发送stream:handleEvent:消息。在打开流之前，我们需要调用流对象的scheduleInRunLoop:forMode:方法，这样做可以避免在没有数据可读时阻塞代理对象的操作。我们需要确保的是流对象被放入正确的run loop中，即放入流事件发生的那个线程的run loop中。

处理流事件

打开流后，我们可以使用streamStatus属性查看流的状态，用hasBytesAvailable属性检测是否有可读的数据，用streamError来查看流处理过程中产生的错误。

流一旦打开后，将会持续发送stream:handleEvent:消息给代理对象，直到流结束为止。这个消息接收一个NSStreamEvent常量作为参数，以标识事件的类型。对于NSInputStream对象，主要的事件类型包括NSStreamEventOpenCompleted、NSStreamEventHasBytesAvailable和NSStreamEventEndEncountered。通常我们会对NSStreamEventHasBytesAvailable更感兴趣。代理清单2演示了从流中获取数据的过程

代理清单2

 
复制代码
- (void)stream:(NSStream *)aStream handleEvent:(NSStreamEvent)eventCode
{
    switch (eventCode) {
        case NSStreamEventHasBytesAvailable:
        {
            if (!data) {
                data = [NSMutableData data];
            }
            uint8_t buf[1024];
            unsigned int len = 0;
            len = [(NSInputStream *)aStream read:buf maxLength:1024];  // 读取数据
            if (len) {
                [data appendBytes:(const void *)buf length:len];
            }
        }
            break;
    }
}


当NSInputStream在处理流的过程中出现错误时，它将停止流处理并产生一个NSStreamEventErrorOccurred事件给代理。我们同样的上面的代理方法中来处理这个事件。

清理流对象

当NSInputStream读取到流的结尾时，会发送一个NSStreamEventEndEncountered事件给代理，代理对象应该销毁流对象，此过程应该与创建时相对应，代码清单3演示了关闭和释放流对象的过程。

代理清单3

 
复制代码
- (void)stream:(NSStream *)aStream handleEvent:(NSStreamEvent)eventCode
{
    switch (eventCode) {
        case NSStreamEventEndEncountered:
        {
            [aStream close];
            [aStream removeFromRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];
            aStream = nil;
        }
            break;
    }
}


写入数据到输出流

类似于从输入流读取数据，写入数据到输出流时，需要下面几个步骤：


使用要写入的数据创建和初始化一个NSOutputStream实例，并设置代理对象
将流对象放到run loop中并打开流
处理流对象发送到代理对象中的事件
如果流对象写入数据到内存，则通过请求NSStreamDataWrittenToMemoryStreamKey属性来获取数据
当没有更多数据可供写入时，处理流对象


基本流程与输入流的读取差不多，我们主要介绍不同的地方

数据可写入的位置包括文件、C缓存、程序内存和网络socket。


hasSpaceAvailable属性表示是否有空间来写入数据
在stream:handleEvent:中主要处理NSStreamEventHasSpaceAvailable事件，并调用流的write:maxLength方法写数据。代码清单4演示了这一过程。
如果NSOutputStream对象的目标是应用的内存时，在NSStreamEventEndEncountered事件中可能需要从内存中获取流中的数据。我们将调用NSOutputStream对象的propertyForKey:的属性，并指定key为NSStreamDataWrittenToMemoryStreamKey来获取这些数据。


代理清单4
- (void)stream:(NSStream *)aStream handleEvent:(NSStreamEvent)eventCode
{
    switch (eventCode) {
        case NSStreamEventHasSpaceAvailable:
        {
            uint8_t *readBytes = (uint8_t *)[data mutableBytes];
            readBytes += byteIndex;
            int data_len = [data length];
            unsigned int len = (data_len - byteIndex >= 1024) ? 1024 : (data_len - byteIndex);
            uint8_t buf[len];

            (void)memcpy(buf, readBytes, len);

            len = [aStream write:(const uint_8 *)buf maxLength:len];
            byteIndex += len;
            break;
        }
    }
}

这里需要注意的是：当代理接收到NSStreamEventHasSpaceAvailable事件而没有写入任何数据到流时，代理将不再从run loop中接收该事件，直到NSOutputStream对象接收到更多数据，这时run loop会重启NSStreamEventHasSpaceAvailable事件。

=================================================


使用NSOutputStream实例需要以下几个步骤：

1，使用存储写入数据的存储库创建和初始化一个NSOutputSteam实例，并且设置它的delegate。

2，将这个流对象布置在一个runloop上并且open the stream。

3，处理流对象向其delegate发送的事件消息。

4，如果流对象向内存中写入了数据，那么可以通过使用NSStreamDataWrittenToMemoryStreamKey属性获取数据。

5，当没有数据可供写入时，清理流对象。



一，使用流对象的准备工作

使用NSOutputStream对象之前你必须指定数据写入的流的目标位置，输出流对象的目标位置可以是file，C buffer， application memory，network socket。

NSOutputStream的初始化方法和工厂方法可以使用a file，a buffer, memory来创建和初始化实例，下面的代码初始化了一个NSOutputStream实例，用来向 application memory 写入数据。

- (void)createOutputStream
{
    NSLog(@"Creating and opening NSOutputStream...");
    // oStream is an instance variable
    oStream = [[NSOutputStream alloc] initToMemory];
    [oStream setDelegate:self];
    [oStream scheduleInRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];
    [oStream open];
}

上面的代码显示，在你初始化一个NSOutputStream对象之后应该设置它的delegate（通常是self），当流对象有 有空间可供数据写入 之类的与流有关的事件消息发送时，delegate会收到从NSOutputStream对象发送来的消息。

当你在open the stream对象之前，向流对象发送scheduleInRunLoop:forMode:消息使其在一个runloop上可以接收到stream events，这样，当流对象不能接收更多数据的时候，可以使delegate避免阻塞。当streaming发生在另外一个线程时，你必须将流对象布置在那个线程的run loop上，You should never attempt to access a scheduled stream from a thread different than the one owning the stream’s run loop. 最后 open the stream 开始数据向 NSOutputStream对象传送。



二，处理 Stream Events

当你向流对象发送open消息之后，你可以通过以下消息获取到流对象的状态，比如说当前是否有空间可供数据写入以及其他错误信息的属性。



streamStatus
hasSpaceAvailable
streamError


返回的状态是NSStreamStatus常量，它指示流当前的状态是opening，writing，at the end of the stream等等，返回的错误是NSError对象，它封装的是所有错误的信息。

重要的是，一旦open the stream，只要delegate持续想流对象写入数据，流对象就是一直向其delegate发送stream:handleEvent:消息，直到到达了流的末尾。这些消息中包含一个NSStreamEvent常量参数来指示事件的类型。对于一个NSOutputStream对象，最常见的事件类型是NSStreamEventOpenCompleted，NSStreamEventHasSpaceAvailable，NSStreamEventEndEncountered，delegate通常对NSStreamEventHasSpaceAvaliable事件最感兴趣。下面的代码就是处理NSStreamEventHasSpaceAvaliable事件的一种方法：


- (void)stream:(NSStream *)stream handleEvent:(NSStreamEvent)eventCode
{
    switch(eventCode)
    {
        case NSStreamEventHasSpaceAvailable:
        {
            uint8_t *readBytes = (uint8_t *)[_data mutableBytes];
            readBytes += byteIndex; // instance variable to move pointer
            int data_len = [_data length];
            unsigned int len = ((data_len - byteIndex >= 1024) ? 1024 : (data_len-byteIndex));
            uint8_t buf[len];
            (void)memcpy(buf, readBytes, len);
            len = [stream write:(const uint8_t *)buf maxLength:len];
            byteIndex += len;
            break;
        }
        // continued ...
    }  
}

在stream:handleEvent:的实现中使用switch语句来判别NSStreamEvent常量，当这个常量是NSStreamEventHasSpacesAvailable的时候，delegate从NSMutableData对象_data中获取数据，并且将其指针转化为适合当前操作的类型u_int8.下一步计算即将进行写操作的字节数（是1024还是所有剩余的字节数），声明一段相应大小的buffer，向该buffer写入相应大小的数据，然后delegate调用流对象write:maxLength:方法将buffer中的数据置入output stream中，最后更新byteIndex用于下一次的读取操作。

如果delegate收到NSStreamEventHasSpacesAvailable事件消息但是没有向stream里写入任何数据，它不会从runloop再接收到space-available的事件消息直到NSOutputStream对象接收到数据，这样由于space-available事件该run loop会重新启动。如果这种情况很有可能在你的程序设计中出现，在收到NSStreamEventHasSpaceAvailable消息并且没有向该stream中写入数据时可以在delegate中设置一个标志位flag，之后，当存在更多的数据需要写入时，先检查该标志位，如果该标志位被设置，那么直接向output-stream实例写入数据。

对于一次向output-stream实例中写入多少数据没有严格的限定，一般情况下使用一些合理值，如512Bytes，1kB，4kB（一个页面大小）。

当在向stream中写数据时NSOutputStream对象发生错误，它会停止streaming并且使用NSStreamEventErrorOccurred消息通知其delegate。



三，清理 Stream Object

当一个NSOutputStream对象结束向一个output stream写入数据，它通过stream:handleEvent:消息向delegate发送NSStreamEventEndEncountered事件消息，这个时候delegate应该清理 stream object，先关闭该stream object，从run loop中移除，释放该stream object。此外，如果NSOutputStream对象的目的存储库是application memory（也就是，你通过initToMemory方法或者其工厂方法outputStreamToMemory创建的该对象），现在就可以从内存中获取数据了。下面的代码实现的清理 stream object的工作：


- (void)stream:(NSStream *)stream handleEvent:(NSStreamEvent)eventCode
{
    switch(eventCode)
    {
        case NSStreamEventEndEncountered:
        {
            NSData *newData = [oStream propertyForKey:NSStreamDataWrittenToMemoryStreamKey];
            if (!newData) {
                NSLog(@"No data written to memory!");
            }
            else {
                [self processData:newData];
            }
            [stream close];
            [stream removeFromRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];
            [stream release];
            oStream = nil; // oStream is instance variable
            break;
        }
        // continued ...
    }  
}

通过向NSOutputStream对象发送propertyForKey:消息获取从流向内存中写入的数据，设定key的值为NSStreamDataWrittenToMemoryStreamKey，该stream object将数据返回到一个NSData对象中。
