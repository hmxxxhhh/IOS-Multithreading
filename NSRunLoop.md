## NSRunLoop
#### 1.概念

1.NSRunLoop是消息机制的处理模式  

NSRunLoop的作用在于有事情做的时候使的当前NSRunLoop的线程工作，没有事情做让当前NSRunLoop的线程休眠  

2.NSRunLoop就是一直在循环检测，从线程start到线程end，检测inputsource(如点击，双击等操作)同步事件，检测timesource同步事件，检测到输入源会执行处理函数，首先会产生通知，corefunction向线程添加runloop observers来监听事件，意在监听事件发生时来做处理。  

#### 2.运行模式
1) NSDefaultRunLoopMode: 默认的运行模式，除了NSConnection对象的事件。  

2) NSRunLoopCommonModes: 是一组常用的模式集合，将一个input source关联到这个模式集合上，等于将input source关联到这个模式集合中的所有模式上。在iOS系统中NSRunLoopCommonMode包含NSDefaultRunLoopMode、NSTaskDeathCheckMode、UITrackingRunLoopMode，我有个timer要关联到这些模式上，一个个注册很麻烦，我可以用CFRunLoopAddCommonMode([[NSRunLoop currentRunLoop] getCFRunLoop],(__bridge CFStringRef) NSEventTrackingRunLoopMode)将NSEventTrackingRunLoopMode或者其他模式添加到这个NSRunLoopCommonModes模式中，然后只需要将Timer关联到NSRunLoopCommonModes，即可以实现Run Loop运行在这个模式集合中任何一个模式时，这个Timer都可以被触发。默认情况下NSRunLoopCommonModes包含了NSDefaultRunLoopMode和UITrackingRunLoopMode。注意：让Run Loop运行在NSRunLoopCommonModes下是没有意义的，因为一个时刻Run Loop只能运行在一个特定模式下，而不可能是个模式集合。  

3) UITrackingRunLoopMode: 用于跟踪触摸事件触发的模式（例如UIScrollView上下滚动），主线程当触摸事件触发时会设置为这个模式，可以用来在控件事件触发过程中设置Timer。  

4) GSEventReceiveRunLoopMode: 用于接受系统事件，属于内部的Run Loop模式。  

5) 自定义Mode：可以设置自定义的运行模式Mode，你也可以用CFRunLoopAddCommonMode添加到NSRunLoopCommonModes中。  

Run Loop运行时只能以一种固定的模式运行，只会监控这个模式下添加的Timer Source和Input Source，如果这个模式下没有相应的事件源，Run Loop的运行也会立刻返回的。注意Run Loop不能在运行在NSRunLoopCommonModes模式，因为NSRunLoopCommonModes其实是个模式集合，而不是一个具体的模式，我可以在添加事件源的时候使用NSRunLoopCommonModes，只要Run Loop运行在NSRunLoopCommonModes中任何一个模式，这个事件源都可以被触发。  

例：  

1.在timer与table同时执行情况，当拖动table时，runloop进入UITrackingRunLoopModes模式下，不会处理定时事件，此时timer不能处理，所以此时将timer加入到NSRunLoopCommonModes模式(addTimer forMode)  

2.在scroll一个页面时来松开，此时connection不会收到消息，由于scroll时runloop为UITrackingRunLoopModes模式，不接收输入源（但是网络IO本事不会影响的，仅仅是网络IO结束后由主线程来处理回调会收到影响），此时要修改connection的mode  

		[connection scheduleInRunLoop:[NSRunLoop currentRunLoop]forMode: NSRunLoopCommonModes];

#### 3.runloop与多线程
一个RunLoop接收两种source的事件：input source和timer source。同时必须知道的是，input source，runloop是异步交付的，而timer source是同步交付的。每个runloop都有一个RunLoop Modes，代表它以何种方式执行。  

在主线程中，系统默认帮我们启动了一个主线程的runloop，并且一直在运行，直到程序退出。而用户创建的子线程，runloop是需要手动启动的，所以在线程里启动timer或者调用performSelector: withObject: afterDelay: inModes: 是需要启动runloop的。  

在主线程用runloop+while阻塞，可以实现和子线程的同步，但是居然神奇的不会卡UI:  

    BOOL threadProcess2Finished =NO;
    - (IBAction)pressActon:(id)sender {
        NSLog(@"Enter buttonRunloopPressed");
        
             threadProcess2Finished =NO;
        
             NSLog(@"Start a new thread.");
        
             [NSThread detachNewThreadSelector: @selector(threadProce2)
                                             toTarget: self
                                           withObject: nil];
        
             while (!threadProcess2Finished) {
                    [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode
                                             beforeDate:[NSDate distantFuture]];
                 }
        
             NSLog(@"Exit buttonRunloopPressed");
         }
    
     -(void)threadProce2{
        
             NSLog(@"Enter threadProce2.");
        
             for (int i=0; i<5;i++) {
            
                     NSLog(@"InthreadProce2 count = %d.", i);
                 
                     sleep(1);
                }
        
            threadProcess2Finished =YES;
         
             NSLog(@"Exit threadProce2.");
         
    }

在子线程中我们这样开启runloop  

            BOOL isRunning = NO;
            do {
                isRunning = [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
            } while (isRunning);

或者这样：

    while (!Done)  
    {
        [[NSRunLoop currentRunLoop] runUntilDate:[NSDate
                    dateWithTimeIntervalSinceNow:10]];
        NSLog(@"exiting runloop.........:");
    }
    //done 默认为no,当事件处理完毕后设置为yes,while退出循环，停止监听

回头看看上篇文章所说的 如果在nsoperation 里直接在当前线程使用connection为什么无法获取数据的问题，这是因为在子线程中runloop默认没有运行的，那么connection自然就无法响应了。

#### 4.何时使用
1.需要用到NSPort或者其他input source跟其他线程通信。  

2.在线程启动timer。  

3.在线程里调用performSelector…这类函数去调用。  

4.让线程执行一个周期性的任务。  

5.NSURLConnection在子线程中发起异步请求  

下面我简单用一个例子怎么在线程里启动timer或者performSelector…如下：  

    -(void)testMain
    {
    　　//开启一个测试子线程
    　　[NSThread detachNewThreadSelector:@selector(threadMethod) toTarget:self withObject:nil];
    }
    
    -(void)threadMethod
    {
    //没用的timer
    
    //NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:.2 target:self selector:@selector(timerDone) userInfo:nil repeats:YES];
    
    //真正启动了timer
    
    NSTimer *timer = [NSTimerscheduledTimerWithTimeInterval:.2target:selfselector:@selector(timerDone) userInfo:nilrepeats:YES];
    
    [[NSRunLoopcurrentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
    
    [[NSRunLoopcurrentRunLoop] run];
    
    　 //同理，调用performSelector也一样
    //[self performSelector:@selector(timerDone) withObject:nil afterDelay:.2];
    //[[NSRunLoop currentRunLoop] run];
    
    }
    -(void)timerDone
    {
    
    NSLog(@”Timer Run“);
    
    }

NStimer，几乎每个做iOS开发的程序员都用过，但是有一个关于Timer的介绍估计很多人都不知道：timer是不一定准时的，是有可能被delay的，每次间隔的时间是不一定一样的。  

A repeating timer reschedules itself automatically based on the scheduled firing time, not the actual firing time. For example, if a timer is scheduled to fire at a particular time and every 5 seconds after that, the scheduled firing time will always fall on the original 5 second time intervals, even if the actual firing time gets delayed. If the firing time is delayed so much that it misses one or more of the scheduled firing times, the timer is fired only once for the missed time period. After firing for the missed period, the timer is rescheduled for the next scheduled firing time.  

简单解读一下：就是说一个repeat的timer，它在创建的时候就把每次的执行时间算好了，而不是真正启动的时候才计算下次的执行时间。举个例子，假如一个timer在一个特定的时间t激活，然后以间隔5秒的时间重复执行。那么它的执行操作的时间就应该为t, t+5, t+10,… 假如它被延迟了，例如实际上timer在t＋2的时候才启动，那么它下次执行的时间还是t＋5，而并不是t＋2+5，如果很不幸地在t＋5还没有启动，那么它理应该在t执行的操作就跟下一次t＋5的操作合为一个了。至于为什么会延迟呢，这就跟当前线程的操作有关，因为timer是同步交付的，所以假如当前线程在执行很复杂的运算时，那必须等待运算的完成才能调用timer，这就导致了timer的延迟。  

我们就用一个例子来看看效果吧，代码为：  

    //这里创建timer以每隔1秒执行
    [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(timerDone) userInfo:nil repeats:YES];
    //这里在第3秒的时候模拟一个复杂运算
    [self performSelector:@selector(busyDone) withObject:nil afterDelay:3];
    -(void)busyDone
    {
    //这里模拟线程复杂的运算
    for(NSInteger i = 0; i< 0xffffffff;i++){
    
    }
    NSLog(@”BusgDone“);
    }
    
    -(void)timerDone
    {
    NSLog(@”Timer Run“);
    }

执行结果为：  

[![](https://github.com/hmxxxhhh/images/blob/master/timer.jpg?raw=true)](https://github.com/hmxxxhhh/images/blob/master/timer.jpg?raw=true)

可以看到，timer本来都是以每隔1秒执行，毫秒都是.564,然后在进行复杂的运算时候，timer直接被delay了，当执行完BusyDone之后，立即执行了TimerRun，然后又在.564执行了TimerRun，而不是距离上次执行时间的1秒。  



