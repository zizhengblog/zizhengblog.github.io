---
layout: post
title: "15、回溯(Back Tracking)"
date: 2020-04-05
description: ""
tag: 数据结构与算法（二）
---






## 目录

* [回溯](#content1)
* [八皇后问题(Eight Queens)](#content2)
* [八皇后问题代码实现](#content3)






<!-- ************************************************ -->
## <a id="content1"></a>回溯

回溯可以理解为：通过选择不同的岔路口来通往目的地（找到想要的结果）     
每一步都选择一条路出发，能进则进，不能进<span style="color:red;font-weight:bold">则退回上一步（回溯）</span>，换一条路再试    
树、图的深度优先搜索（DFS）、八皇后、走迷宫都是典型的回溯应用

<img src="/images/DataStructurs2/backtracking1.png" alt="img">

<span style="font-weight:bold">不难看出来，回溯很适合使用递归</span>


<!-- ************************************************ -->
## <a id="content2"></a>八皇后问题(Eight Queens)

**一、问题**

八皇后问题是一个古老而著名的问题     
在8x8格的国际象棋上摆放八个皇后，使其不能互相攻击：任意两个皇后都不能处于同一行、同一列、同一斜线上,请问有多少种摆法？     
 
<img src="/images/DataStructurs2/backtracking2.png" alt="img">

**二、思路**

思路一：暴力出奇迹     
从 64 个格子中选出任意 8 个格子摆放皇后，检查每一种摆法的可行性     
一共 C8/64 种摆法（大概是 4.4 * 10^9 种摆法）    


思路二：根据题意减小暴力程度     
很显然，每一行只能放一个皇后，所以共有 8^8 种摆法（16777216 种），检查每一种摆法的可行性

思路三：回溯法     
回溯 + 剪枝
 
**三、回溯 + 剪枝**

在解决八皇后问题之前，可以先缩小数据规模，看看如何解决四皇后问题
 
<img src="/images/DataStructurs2/backtracking3.png" alt="img">

<center style="font-weight:bold">剪枝</center>
<img src="/images/DataStructurs2/backtracking4.png" alt="img">

**四、八皇后分析**
<img src="/images/DataStructurs2/backtracking5.png" alt="img">
<img src="/images/DataStructurs2/backtracking6.png" alt="img">
<img src="/images/DataStructurs2/backtracking7.png" alt="img">


<!-- ************************************************ -->
## <a id="content3"></a>八皇后问题代码实现

**一、实现思路一**

```
@interface LCQueens : NSObject
/// N皇后问题
/// @param n 皇后个数
-(void)placeQueens:(int)n;
@end
```
```
#import "LCQueens.h"

@interface LCQueens()
@property(nonatomic,assign)int ways;
@property(nonatomic,strong)NSMutableArray<NSNumber*>* cols;//索引作为行号，元素作为列号
@end

@implementation LCQueens

- (NSMutableArray<NSNumber *> *)cols{
    if (_cols == nil) {
        _cols = [NSMutableArray array];
    }
    return _cols;
}

/// N皇后问题
/// @param n 皇后个数
-(void)placeQueens:(int)n{
    if (n < 1) return;
    
    //初始化cols
    for (int i= 0; i<n; i++) {
        self.cols[i] = @(0);
    }
    
    [self __place:0];
    
     printf("%d 皇后一共有 %d 种摆法",n,self.ways);
    self.ways = 0;
}


/// 从第row行开始摆放皇后
/// @param row 行
-(void)__place:(int)row{
    
    //最后一行已经摆完，摆放成功
    if (row == self.cols.count) {
        self.ways++;
        [self __show];
        return;
    }
    
    //n = self.cols.count
    for (int col = 0; col < self.cols.count; col++) {
        //row:col 是否可以放置
        if ([self __isValidRow:row col:col]) {
            //放置成功
            self.cols[row] = @(col);
            
            //放置下一行
            [self __place:row+1];
            
            //来到此处：下一行未放置成功
        }else{
            //来到此处：该行该位置不可放置
        }
        
        //来到此处：在该行寻找下一位置
    }
    
    //来到此处：在该行没有找到可以放置的位置，函数调用结束，返回上次调用（回溯）
}

-(BOOL)__isValidRow:(int)row col:(int)col{
    
    for (int i = 0; i<row; i++) {
        //第col列已经有皇后
        if ([self.cols[i] intValue] == col) {
//            NSLog(@"[%d][%d]=false",row,col);
            return false;
        }
        
        // 第i行的皇后跟第row行第col列格子处在同一斜线上
        if (row-i == abs(col - [self.cols[i] intValue])) {
//            NSLog(@"[%d][%d]=false",row,col);
            return false;
        }
    }
    
//    NSLog(@"[%d][%d]=true",row,col);
    return true;
}



-(void)__show{
    for (int row = 0; row < self.cols.count; row++) {
        for (int col = 0; col < self.cols.count; col++) {
            if ([self.cols[row] intValue] == col) {
                printf("1 ");
            } else {
                printf("0 ");
            }
        }
         printf("\n");
    }
     printf("------------------------------\n");
}



@end

```

**二、实现思路二**

```
@interface LCQueens : NSObject
/// N皇后问题
/// @param n 皇后个数
-(void)placeQueens:(int)n;
@end
```


```
#import "LCQueens2.h"

@interface LCQueens2()
@property(nonatomic,assign)int ways;
@property(nonatomic,strong)NSMutableArray* cols;    //记录某一列是否有皇后
@property(nonatomic,strong)NSMutableArray* leftTop; //记录左上右下对角线是否有皇后
@property(nonatomic,strong)NSMutableArray* rightTop;//记录右上左下对角线是否有皇后
@property(nonatomic,strong)NSMutableArray* queens;  //记录皇后的位置，索引行号，元素列号
@end

@implementation LCQueens2

-(NSMutableArray *)cols{
    if (_cols == nil) {
        _cols = [NSMutableArray array];
    }
    return _cols;
}

-(NSMutableArray *)leftTop{
    if (_leftTop == nil) {
        _leftTop = [NSMutableArray array];
    }
    return _leftTop;
}

-(NSMutableArray *)rightTop{
    if (_rightTop == nil) {
        _rightTop = [NSMutableArray array];
    }
    return _rightTop;
}

-(NSMutableArray *)queens{
    if (_queens == nil) {
        _queens = [NSMutableArray array];
    }
    return _queens;
}


/// N皇后问题
/// @param n 皇后个数
-(void)placeQueens:(int)n{
    if (n < 1) return;
    
    //初始化cols、leftTop、rightTop、Queens
    [self __initArray:self.cols n:n];
    [self __initArray:self.leftTop n:(n<<1)-1];
    [self __initArray:self.rightTop n:(n<<1)-1];
    [self __initArray:self.queens n:n];
    
    [self __place:0];

    printf("%d 皇后一共有 %d 种摆法",n,self.ways);
    self.ways = 0;
}


/// 从第row行开始摆放皇后
/// @param row 行
-(void)__place:(int)row{
    
    //最后一行已经摆完，摆放成功
    if (row == self.cols.count) {
        self.ways++;
        [self __show];
        return;
    }
    
    //n = self.cols.count
    for (int col = 0; col < self.cols.count; col++) {

        //row:col 是否可以放置
        if ([self.cols[col] intValue]) continue;;
        
        int leftTopIndex  =  row - col + (int)self.cols.count - 1;
        if ([self.leftTop[leftTopIndex] intValue]) continue;
        
        int rightTopIndex = row + col;
        if ([self.rightTop[rightTopIndex] intValue]) continue;

        
        //记录该列已经有皇后
        self.cols[col] = @(1);
        
        //记录该对角线有皇后
        self.leftTop[leftTopIndex] = @(1);
        
        //记录该对角线有皇后
        self.rightTop[rightTopIndex] = @(1);
        
        self.queens[row] = @(col);
        
        
        //放置下一行
        [self __place:row+1];
        
        
        //来到此处：下一行放置失败还原数据：寻找下一个位置
        self.cols[col] = @(0);
        self.leftTop[leftTopIndex] = @(0);
        self.rightTop[rightTopIndex] = @(0);
    }
    
    //来到此处：在该行没有找到可以放置的位置，函数调用结束，返回上次调用（回溯）
}





-(void)__show{
    for (int row = 0; row < self.queens.count; row++) {
        for (int col = 0; col < self.queens.count; col++) {
            if ([self.queens[row] intValue] == col) {
                printf("1 ");
            } else {
                printf("0 ");
            }
        }
         printf("\n");
    }
     printf("------------------------------\n");
}




/// 将数组初始化为@(0)
/// @param array 数组
/// @param n 个数
-(void)__initArray:(NSMutableArray*)array n:(int)n{
    for (int i = 0; i<n; i++) {
        array[i] = @(0);
    }
}

@end
```

**三、实现思路三**

```
@interface LCQueens : NSObject
/// N皇后问题
/// @param n 皇后个数
-(void)placeQueens:(int)n;
@end
```

```
#import "LCQueens3.h"

//这种优化只适用于4皇后和八皇后问题；因为Byte 和 short 决定了bit位的数量
//如果想要解决更大的皇后问题，请更改Byte 和 short的数据类型比如：long、long long
//更大的可以使用数组，但数组的使用是一个bit一个bit的使用而不是常规使用
@interface LCQueens3()
@property(nonatomic,assign)int queenCount;
@property(nonatomic,assign)int ways;
@property(nonatomic,assign)Byte cols;    //记录某一列是否有皇后
@property(nonatomic,assign)short leftTop; //记录左上右下对角线是否有皇后
@property(nonatomic,assign)short rightTop;//记录右上左下对角线是否有皇后
@property(nonatomic,strong)NSMutableArray* queens;  //记录皇后的位置，索引行号，元素列号
@end

@implementation LCQueens3

-(NSMutableArray *)queens{
    if (_queens == nil) {
        _queens = [NSMutableArray array];
    }
    return _queens;
}

/// N皇后问题
/// @param n 皇后个数
-(void)placeQueens:(int)n{
    if (n < 1) return;
    self.queenCount = n;

    [self __place:0];

    printf("%d 皇后一共有 %d 种摆法",n,self.ways);
    self.ways = 0;
}


/// 从第row行开始摆放皇后
/// @param row 行
-(void)__place:(int)row{
    
    //最后一行已经摆完，摆放成功
    if (row == self.queenCount) {
        self.ways++;
        [self __show];
        return;
    }
    
    //n = self.cols.count
    for (int col = 0; col < self.queenCount; col++) {

        //row:col 是否可以放置
        if (self.cols & (1<<col)) continue;;
        
        int leftTopIndex  =  row - col + self.queenCount - 1;
        if (self.leftTop & (1<<leftTopIndex)) continue;
        
        int rightTopIndex = row + col;
        if (self.rightTop & (1<<rightTopIndex)) continue;

        
        //记录该列已经有皇后
        self.cols |= 1<<col;
        
        //记录该对角线有皇后
        self.leftTop |= 1<<leftTopIndex;
        
        //记录该对角线有皇后
        self.rightTop |= 1<<rightTopIndex;
        
        self.queens[row] = @(col);
        
        
        //放置下一行
        [self __place:row+1];
        
        
        //来到此处：下一行放置失败还原数据：寻找下一个位置
        self.cols &= ~(1<<col);
        self.leftTop &= ~(1<<leftTopIndex);
        self.rightTop &= ~(1<<rightTopIndex);
    }
    
    //来到此处：在该行没有找到可以放置的位置，函数调用结束，返回上次调用（回溯）
}





-(void)__show{
    for (int row = 0; row < self.queens.count; row++) {
        for (int col = 0; col < self.queens.count; col++) {
            if ([self.queens[row] intValue] == col) {
                printf("1 ");
            } else {
                printf("0 ");
            }
        }
         printf("\n");
    }
     printf("------------------------------\n");
}



@end
```



----------
>  行者常至，为者常成！


