---
layout: post
title: "NSOperation"
date: 2018-08-05
description: "NSOperation"
tag: Objective-C
---





 



## 目录


* [同步](#content0)
* [异步](#content1)
* [其它](#content2)





NSOperation 底层是GCD,比GCD多了一些简单实用的功能，使用更加面向对象。


<!-- ************************************************ -->
## <a id="content0"></a>同步
在当前线程执行任务
```objc
//invocationOperation
NSInvocationOperation * operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(_test) object:nil];
[operation start];
```

```objc
//blockOperation
__weak typeof(self) weakSeal = self;
NSBlockOperation * blockOperation = [NSBlockOperation blockOperationWithBlock:^{
    [weakSeal _test];
}];
[blockOperation start];
```



<!-- ************************************************ -->
## <a id="content1"></a>异步
1.异步使用   
```objc
//异步并发队列的封装
NSOperationQueue * concurrentQueue = [[NSOperationQueue alloc] init];

//创建一个操作任务
NSInvocationOperation * operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(_test) object:nil];

//添加到队列后 会直接开启新线程执行
[concurrentQueue addOperation:operation];
```

2.队列的最大并发数
```objc
//最大并发数
NSOperationQueue * concurrentQueue = [[NSOperationQueue alloc] init];
//concurrentQueue.maxConcurrentOperationCount = 1;//最大并发数为1 可同时执行1个任务 串行队列
concurrentQueue.maxConcurrentOperationCount = 2;//最大并发数为2 可同时执行2个任务
//concurrentQueue.maxConcurrentOperationCount = 3;//最大并发数为3 可同时执行3个任务

//添加到队列里就会执行
[concurrentQueue addOperationWithBlock:^{
    NSLog(@"block1-耗时操作");
}];

[concurrentQueue addOperationWithBlock:^{
    NSLog(@"block2-耗时操作");
}];

[concurrentQueue addOperationWithBlock:^{
    NSLog(@"block3-耗时操作");
}];

//PS：重要说明
//最大并发数并不是指开辟线程的数量，而是指同时执行的操作数的数量
```

3.队列的挂起
```objc
-(void)operation_suspend{
    NSOperationQueue * concurrentQueue = [[NSOperationQueue alloc] init];
    [concurrentQueue addOperationWithBlock:^{
        NSLog(@"lilog - 耗时操作");
    }];
    concurrentQueue.suspended = YES;//挂起队列
    concurrentQueue.suspended = NO;//取消队列挂起
}
```

4.队列的取消
```objc
//队列的取消
-(void)operation_cancel{
    NSOperationQueue * concurrentQueue = [[NSOperationQueue alloc] init];
    [concurrentQueue addOperationWithBlock:^{
        NSLog(@"lilog - 耗时操作");
    }];
    [concurrentQueue cancelAllOperations];//取消队列
}
```


<!-- ************************************************ -->
## <a id="content2"></a>其它

1.子线程回主线程
```objc
//线程间通讯
NSOperationQueue * concurrentQueue = [[NSOperationQueue alloc] init];
[concurrentQueue addOperationWithBlock:^{
    NSLog(@"lilog - 耗时操作");
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        NSLog(@"lilog - 回到主线程-%@",[NSThread currentThread]);
    }];
}];
```

2.操作任务间的依赖关系
```objc
-(void)operation_dependency{
    NSBlockOperation * operation1 = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i<10; i++) {
            NSLog(@"thread = %@ i = %d",[NSThread currentThread],i);
        }
    }];
    
    NSBlockOperation * operation2 = [NSBlockOperation blockOperationWithBlock:^{
        for (int j = 0; j<10; j++) {
            NSLog(@"thread = %@ j = %d",[NSThread currentThread],j);
        }
    }];
    
    NSBlockOperation * operation3 = [NSBlockOperation blockOperationWithBlock:^{
        for (int k = 0; k<10; k++) {
            NSLog(@"thread = %@ k = %d",[NSThread currentThread],k);
        }
        
    }];
    
    //操作之间的执行关系  切记：不可添加循环的依赖关系
    [operation2 addDependency:operation1];//operation2 需要在 operation1执行完之后再执行
    [operation3 addDependency:operation2];//operation3 需要在 operation2执行完之后再执行
    
    //创建一个队列
    NSOperationQueue * queue = [[NSOperationQueue alloc] init];
    [queue addOperation:operation1];
    [queue addOperation:operation2];
    [queue addOperation:operation3];
    
    //此代码会保证queue内所有的任务都执行完之后再执行其后代码
    [queue waitUntilAllOperationsAreFinished];
    NSLog(@"come here");
}
```



----------
>  行者常至，为者常成！


