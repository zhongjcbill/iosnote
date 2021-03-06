# NSOperation

##NSOperation的作用
- 配合使用NSOperation和NSOperationQueue也能实现多线程编程

##使用NSOperation子类的方式有3种
- NSInvocationOperation
- NSBlockOperation
- 自定义子类继承NSOperation，实现内部相应的方法

##NSInvocationOperation
- 注意
- 默认情况下，调用了start方法后并不会开一条新线程去执行操作，而是在当前线程同步执行操作
- 只有将NSOperation放到一个NSOperationQueue中，才会异步执行操作

```objc
- (void)test2
{
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(print:) object:@"hello"];
    
    //不会开新线程
    [op start];
}


log 
-NSOperationQueue[1600:268010] ----------print,hello
-NSOperationQueue[1600:268010] <NSThread: 0x7fb380f00ac0>{number = 1, name = main}
```

- 加入队列会开启新线程

```objc
- (void)test2
{
    
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(print:) object:@"hello"];
    
    [queue addOperation:op];
}

- (void)print:(NSString *)argv
{
    NSLog(@"----------print,%@",argv);
    NSLog(@"%@",[NSThread currentThread]);
}

log 打印
掌握-NSOperationQueue[1614:271415] ----------print,hello
掌握-NSOperationQueue[1614:271415] <NSThread: 0x7fc610e78a90>{number = 2, name = (null)}
```

##NSBlockOperation
```objc
//创建NSBlockOperation对象
+ (id)blockOperationWithBlock:(void (^)(void))block;

//通过addExecutionBlock:方法添加更多的操作
- (void)addExecutionBlock:(void (^)(void))block;
```

- 注意：只要NSBlockOperation封装的操作数 > 1，就会异步执行操作

```objc
- (void)test3
{
       
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
                NSLog(@"----------NSBlockOperation1");
                NSLog(@"%@",[NSThread currentThread]);
            }];
        
    [op addExecutionBlock:^{
        NSLog(@"----------NSBlockOperation2");
        NSLog(@"%@",[NSThread currentThread]);
    }];
    
    [op addExecutionBlock:^{
        NSLog(@"----------NSBlockOperation3");
        NSLog(@"%@",[NSThread currentThread]);
    }];
    
    [op start];

log 打印
掌握-NSOperationQueue[1651:279652] ----------NSBlockOperation1
掌握-NSOperationQueue[1651:279652] <NSThread: 0x7fcae3505d60>{number = 1, name = main}
掌握-NSOperationQueue[1651:279706] ----------NSBlockOperation2
掌握-NSOperationQueue[1651:279713] ----------NSBlockOperation3
掌握-NSOperationQueue[1651:279706] <NSThread: 0x7fcae360c250>{number = 2, name = (null)}
掌握-NSOperationQueue[1651:279713] <NSThread: 0x7fcae3712400>{number = 3, name = (null)}
}
```

##NSOperationQueue
- NSOperation可以调用start方法来执行任务，但默认是同步执行的
- 如果将NSOperation添加到NSOperationQueue（操作队列）中，系统会自动异步执行NSOperation中的操作

```objc
添加操作到NSOperationQueue中
- (void)addOperation:(NSOperation *)op;
- (void)addOperationWithBlock:(void (^)(void))block;
```

##最大并发数

- 当队列最大并发数为1时，表示串行

```objc
//什么是并发数:比如同时开3个线程执行3个任务，并发数就是3
//最大并发数的相关方法
- (NSInteger)maxConcurrentOperationCount;
- (void)setMaxConcurrentOperationCount:(NSInteger)cnt;
```

```objc
- (void)test1
{
     
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    //最大并发数
    queue.maxConcurrentOperationCount = 1;
    
    [queue addOperationWithBlock:^{
        NSLog(@"----------NSBlockOperation1");
        NSLog(@"%@",[NSThread currentThread]);

    }];
    
    [queue addOperationWithBlock:^{
        NSLog(@"----------NSBlockOperation2");
        NSLog(@"%@",[NSThread currentThread]);
        
    }];
    
    [queue addOperationWithBlock:^{
        NSLog(@"----------NSBlockOperation3");
        NSLog(@"%@",[NSThread currentThread]);
        
    }];
    
    [queue addOperationWithBlock:^{
        NSLog(@"----------NSBlockOperation4");
        NSLog(@"%@",[NSThread currentThread]);
        
    }];
    
 log 打印
-NSOperationQueue[1666:287432] ----------NSBlockOperation1
-NSOperationQueue[1666:287432] <NSThread: 0x7fba515134b0>{number = 2, name = (null)}
-NSOperationQueue[1666:287432] ----------NSBlockOperation2
-NSOperationQueue[1666:287432] <NSThread: 0x7fba515134b0>{number = 2, name = (null)}
-NSOperationQueue[1666:287432] ----------NSBlockOperation3
-NSOperationQueue[1666:287432] <NSThread: 0x7fba515134b0>{number = 2, name = (null)}
-NSOperationQueue[1666:287432] ----------NSBlockOperation4
-NSOperationQueue[1666:287432] <NSThread: 0x7fba515134b0>{number = 2, name = (null)}
}
```

##队列的取消、暂停、恢复
```objc
//取消队列的所有操作
- (void)cancelAllOperations;
提示：也可以调用NSOperation的- (void)cancel方法取消单个操作

//暂停和恢复队列
- (void)setSuspended:(BOOL)b; // YES代表暂停队列，NO代表恢复队列
- (BOOL)isSuspended;
```

## 操作的监听
```objc
//可以监听一个操作的执行完毕
- (void (^)(void))completionBlock;
- (void)setCompletionBlock:(void (^)(void))block;
```

```objc
- (void)test2
{
    
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(print:) object:@"hello"];
    
    [op setCompletionBlock:^{
        NSLog(@"完了");
    }];
    
    [queue addOperation:op];
}

- (void)print:(NSString *)argv
{
    NSLog(@"----------print,%@",argv);
    NSLog(@"%@",[NSThread currentThread]);
}

log 打印
掌握-NSOperationQueue[1680:294419] ---------- print,hello
掌握-NSOperationQueue[1680:294419] <NSThread: 0x7fd37942f640>{number = 2, name = (null)}
掌握-NSOperationQueue[1680:294423] 完了
```

##操作依赖
- NSOperation之间可以设置依赖来保证执行顺序
- 比如一定要让操作A执行完后，才能执行操作B，可以这么写[operationB addDependency:operationA]; // 操作B依赖于操作A
- 可以在不同queue的NSOperation之间创建依赖关系

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"download1----%@", [NSThread  currentThread]);
    }];
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"download2----%@", [NSThread  currentThread]);
    }];
    NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"download3----%@", [NSThread  currentThread]);
    }];
    NSBlockOperation *op4 = [NSBlockOperation blockOperationWithBlock:^{
        for (NSInteger i = 0; i<10; i++) {
            NSLog(@"download4----%@", [NSThread  currentThread]);
        }
    }];
    NSBlockOperation *op5 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"download5----%@", [NSThread  currentThread]);
    }];
    op5.completionBlock = ^{
        NSLog(@"op5执行完毕---%@", [NSThread currentThread]);
    };
    
    // 设置依赖
    [op3 addDependency:op1];
    [op3 addDependency:op2];
    [op3 addDependency:op4];
    
    [queue addOperation:op1];
    [queue addOperation:op2];
    [queue addOperation:op3];
    [queue addOperation:op4];
    [queue addOperation:op5];
}

log 打印
2016-07-07 16:35:45.260 04-掌握-操作依赖[1715:299708] download1----<NSThread: 0x7f9f61e1c040>{number = 2, name = (null)}
2016-07-07 16:35:45.260 04-掌握-操作依赖[1715:299697] download2----<NSThread: 0x7f9f61f0ec20>{number = 3, name = (null)}
2016-07-07 16:35:45.260 04-掌握-操作依赖[1715:299747] download5----<NSThread: 0x7f9f61c1a990>{number = 4, name = (null)}
2016-07-07 16:35:45.260 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.261 04-掌握-操作依赖[1715:299747] op5执行完毕---<NSThread: 0x7f9f61c1a990>{number = 4, name = (null)}
2016-07-07 16:35:45.261 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.261 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.262 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.262 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.262 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.262 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.262 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.263 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.303 04-掌握-操作依赖[1715:299706] download4----<NSThread: 0x7f9f61f13fb0>{number = 5, name = (null)}
2016-07-07 16:35:45.303 04-掌握-操作依赖[1715:299708] download3----<NSThread: 0x7f9f61e1c040>{number = 2, name = (null)}

```


