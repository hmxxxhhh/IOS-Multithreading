# iOS多线程介绍篇
## GCD
GCD编程的核心就是dispatch队列，dispatch block的执行最终都会放进某个队列中去进行，它类似NSOperationQueue但更复杂也更强大，并且可以嵌套使用。所以说，结合block实现的GCD，把函数闭包（ Closure ）的特性发挥得淋漓尽致。  

**dispatch队列的生成可以有这几种方式：**   
##### 1.
    dispatch_queue_t queue = dispatch_queue_create ( "com.dispatch.serial" , DISPATCH_QUEUE_SERIAL );

生成一个串行队列，队列中的block按照先进先出（FIFO）的顺序去执行，实际上为单线程执行。第一个参数是队列的名称，在调试程序时会非常有用，所有尽量不要重名了。  
##### 2.
     dispatch_queue_t  queue =  dispatch_queue_create ( "com.dispatch.concurrent" ,  DISPATCH_QUEUE_CONCURRENT );

生成一个并发执行队列，block被分发到多个线程去执行。  
##### 3.
    dispatch_queue_t  queue =  dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); 

**获得程序进程缺省产生的并发队列**，可设定优先级来选择高、中、低三个优先级队列。由于是系统默认生成的，所以无法调用dispatch_resume()和dispatch_suspend()来控制执行继续或中断。需要注意的是，三个队列不代表三个线程，可能会有更多的线程。并发队列可以根据实际情况来自动产生合理的线程数，也可理解为dispatch队列实现了一个线程池的管理，对于程序逻辑是透明的。  

官网文档解释说共有三个并发队列，但实际还有一个更低优先级的队列，设置优先级为 DISPATCH_QUEUE_PRIORITY_BACKGROUND 。Xcode调试时可以观察到正在使用的各个dispatch队列。   
##### 4.
    dispatch_queue_t  queue =  dispatch_get_main_queue();

获得主线程的dispatch队列，实际是一个串行队列。同样无法控制主线程dispatch队列的执行继续或中断。  

**接下来我们可以使用dispatch_async或dispatch_sync函数来加载需要运行的block**  

    dispatch_async(queue, ^{
    
    //block具体代码
    
    }); //异步执行block，函数立即返回
    
    dispatch_sync(queue, ^{
    
    //block具体代码
    
    }); 

同步执行block，函数不返回，一直等到block执行完毕。编译器会根据实际情况优化代码，所以有时候你会发现block其实还在当前线程上执行，并没用产生新线程。  

实际编程经验告诉我们，尽可能避免使用dispatch_sync，嵌套使用时还容易引起程序死锁。  

如果queue1是一个串行队列的话，这段代码立即产生死锁：

    dispatch_sync (queue1, ^{
    
          dispatch_sync (queue1, ^{
    
    ......
    
    });
    
    ......
    
    });
下面这段也会产生死锁：

    dispatch_queue_t queue1 = dispatch_queue_create("com.liancheng.serial_queue", DISPATCH_QUEUE_SERIAL);
        
        dispatch_async(queue1, ^{
            // 到达串行队列
            NSLog(@"11");
            dispatch_sync(queue1, ^{     //发生死锁
                NSLog(@"dd");
            });
            NSLog(@"33");
        });

那实际运用中，一般可以用dispatch这样来写，常见的网络请求数据多线程执行模型：  


    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    
    //子线程中开始网络请求数据
    
    //更新数据模型
    
    dispatch_sync(dispatch_get_main_queue(), ^{
    
    //在主线程中更新UI代码
    
    });
    
    });

dispatch队列是线程安全的，可以利用串行队列实现锁的功能。比如多线程写同一数据库，需要保持写入的顺序和每次写入的完整性，简单地利用串行队列即可实现：  


    dispatch_queue_t  queue1 =  dispatch_queue_create ( "com.dispatch.writedb" ,  DISPATCH_QUEUE_SERIAL );
    
    - (void)writeDB:(NSData *)data
    
    {
    
    dispatch_sync(queue1, ^{
    
    //write database
    
    });
    
    } 

下一次调用writeDB:必须等到上次调用完成后才能进行，保证writeDB:方法是线程安全的。  

**dispatch队列还实现其它一些常用函数，包括：**  

    void dispatch_apply( size_t iterations, dispatch_queue_t queue,  void (^block)( size_t )); 

重复执行block，需要注意的是这个方法是同步返回，也就是说等到所有block执行完毕才返回，如需异步返回则嵌套在dispatch_async中来使用。多个block的运行是否并发或串行执行也依赖queue的是否并发或串行。  

    void dispatch_barrier_async( dispatch_queue_t queue, dispatch_block_t block); 

这个函数可以设置同步执行的block，它会等到在它加入队列之前的block执行完毕后，才开始执行。在它之后加入队列的block，则等到这个block执行完毕后才开始执行。  

    void dispatch_barrier_sync( dispatch_queue_t  queue,  dispatch_block_t  block);

同上，除了它是同步返回函数。  

    void dispatch_after( dispatch_time_t when, dispatch_queue_t queue,  dispatch_block_t block); 

延迟执行block。  

**最后再来看看dispatch队列的一个很有特色的函数：**  

    void dispatch_set_target_queue( dispatch_object_t object, dispatch_queue_t queue);

它会把需要执行的任务对象指定到不同的队列中去处理，这个任务对象可以是dispatch队列，也可以是dispatch源（以后博文会介绍）。而且这个过程可以是动态的，可以实现队列的动态调度管理等等。比如说有两个队列dispatchA和dispatchB，这时把dispatchA指派到dispatchB：  

`dispatch_set_target_queue(dispatchA, dispatchB);`

那么dispatchA上还未运行的block会在dispatchB上运行。这时如果暂停dispatchA运行：  

`dispatch_suspend(dispatchA);`

则只会暂停dispatchA上原来的block的执行，dispatchB的block则不受影响。而如果暂停dispatchB的运行，则会暂停dispatchA的运行。  

这里只简单举个例子，说明dispatch队列运行的灵活性，在实际应用中你会逐步发掘出它的潜力。  

## NSThread
#### 一、NSthread的初始化  
1.动态方法  

    // 初始化线程  
    NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];  
    // 设置线程的优先级(0.0 - 1.0，1.0最高级)  
    thread.threadPriority = 1;  
    // 开启线程  
    [thread start];

2.静态方法  

    [NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:nil];  
    // 调用完毕后，会马上创建并开启新线程  

3.隐式创建线程的方法  

    [self performSelectorInBackground:@selector(run) withObject:nil];  

#### 二、获取线程  

    NSThread *current = [NSThread currentThread]; 
	NSThread *main = [NSThread mainThread]; 

#### 三、暂停当前线程  

    // 暂停2s  
    [NSThread sleepForTimeInterval:2];  
      
    // 或者  
    NSDate *date = [NSDate dateWithTimeInterval:2 sinceDate:[NSDate date]];  
    [NSThread sleepUntilDate:date];

#### 四、线程间通信  

    //在指定线程上执行操作
    [self performSelector:@selector(run) onThread:thread withObject:nil waitUntilDone:YES];  
    //在主线程上执行操作
    [self performSelectorOnMainThread:@selector(run) withObject:nil waitUntilDone:YES]; 
    //在当前线程执行操作
    [self performSelector:@selector(run) withObject:nil];  

## NSOperation 

##### 默认情况下，NSOperation并不具备封装操作的能力，必须使用它的子类，使用NSOperation子类的方式有3种：    
a、 自定义子类继承NSOperation，实现内部相应的方法   
b、NSBlockOperation  
c、 NSInvocationOperation   
##### 2、NSOperation
1、NSOperation可以用来封装并发或非并发的操作  

 2、对于非并发的操作可以直接重写main方法（main方法里不开启多线程），这时候如果调用start方法，那么main里面的代码是同步执行的即会阻塞主线程。但是如果加入到 queue 中的话，queue会为没个operation分配一个线程，此时是异步执行的不会阻塞主线程。  
 
  3，对于并发的操作，你可以重写start方法在其中用GCD或Thread或NSURLConnection来实现多线程，特别注意如果你用NSURLConnection用代理的方式接受数据的时候，你必须在主线程中重新为connection指定线程，不然代理无法接受到数据（至于原因参考下篇NSRunLoop的文章），你可以这样写：  
  
      dispatch_async(dispatch_get_main_queue(), ^{
          self.connection = [[NSURLConnection alloc] initWithRequest:self.request
                                                            delegate:self
                                                    startImmediately:NO];
          [self.connection scheduleInRunLoop:[NSRunLoop currentRunLoop]
                                     forMode:NSRunLoopCommonModes];
          [self.connection start];
        });
或者这样参考AF的写法：  
    - (void)start {
        [self.lock lock];
        if ([self isCancelled]) {
            [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
        } else if ([self isReady]) {
            self.state = AFOperationExecutingState;
    
            [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
        }
        [self.lock unlock];
    }
只是这样写程序虽然能运行，但并不完整，应为这个时候，operation的isFinished，isExecuting，isReady，isAsynchronous，isConcurrent状态都是不对的，当用queue管理的时候，你如果设置依赖，优先级等都会无效，所有你应该重写这几个方法以确保queue能正常管理。  

##### 2、NSBlockOperation
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
##### 3、NSInvocationOperation
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
