# WeChatLuckyMoney
微信（WeChat）随机红包金额分配算法 ，这里提供两种可能的算法（以整型计算）

## 第一种

**基本思路**：根据剩余金额和剩余人数，求出平均值（即剩余人分得红包的近似期望值）。在 [0，平均值 * 2] 之间产生随机数，判定最小值为 1，这个随机数即为一次随机的金额。剩余金额减去这个数，继续重复上面的步骤，直到剩最后一人时，把所剩全部金额给他。

```objective-c
- (NSArray<NSNumber *> *)randomRedPacketsWithMoney:(int)totalMoney size:(int)size {
    
    /**** 本计算的单位为"分" ****/
    NSMutableArray<NSNumber *> *redPackets = [[NSMutableArray alloc] init];
    
    // 剩余金额
    int remainMoney = totalMoney;
    // 剩余数量
    int remainSize  = size;
    
    while (remainSize > 0) {
        
        // 最后一个红包
        if (remainSize == 1) {
            remainSize --;
            [redPackets addObject:@(remainMoney)];
            return [redPackets copy];
        }
        
        // 其他红包
        int min     = 1; // 最低金额 1分
        int average = remainMoney /remainSize;
        int money   = arc4random_uniform(average * 2);
        
        money = money > min ? money : min;
        [redPackets addObject:@(money)];
        
        remainSize  --;
        remainMoney -= money;
        
    }
    return [redPackets copy];
}
```

这种算法，思路简单，运算速度快，可以进行实时动态计算红包金额。

但是红包的金额数不会超过剩余平均值的二倍。后领取的红包金额方差逐渐变大，红包金额也有略微提高

```objective-c
// 2000 的总额度，分配给 20 个单位，重复执行 100万 次，对应 index 元素相加的结果
(
    99531667,
    99063808,
    99035335,
    99143978,
    99175351,
    99376282,
    99288965,
    99404003,
    99628620,
    99622119,
    99654040,
    99884774,
    99912552,
    100090908,
    100223858,
    100501655,
    100800964,
    101040741,
    101448313,
    103172067
)

// 总用时 5.868397112004459 (Mac上执行)
```

## 第二种

**基本思路**：随机产生 `size - 1` 个数，按照小到大排序，按比例将 `totalMoney` 分成 `size` 份，每一份既是一个随机红包。

```objective-c
- (NSArray<NSNumber *> *)newRandomRedPacketsWithMoney:(int)totalMoney size:(int)size {
    
    /**** 本计算的单位为"分" ****/
    NSMutableArray<NSNumber *> *cutNumbers = [[NSMutableArray alloc] init];
    
    int min = 1;
    int randomMoney = totalMoney - size * min;
    for (int i = 0; i + 1 < size; i++) {
        int money = arc4random_uniform(randomMoney);
        [cutNumbers addObject:@(money)];
    }
    
    [cutNumbers sortUsingComparator:^NSComparisonResult(NSNumber *obj1, NSNumber *obj2) {
        return obj1.intValue > obj2.intValue;
    }];
    
    /**** 红包数组 ****/
    NSMutableArray<NSNumber *> *redPackets = [[NSMutableArray alloc] init];
    
    for (int i = 0; i < cutNumbers.count; i++) {
        int money = 0;
        if (i == 0) {
            money = cutNumbers[i].intValue + min;
        } else {
            money = cutNumbers[i].intValue - cutNumbers[i - 1].intValue + min;
        }
        [redPackets addObject:@(money)];
    }
    [redPackets addObject:@(randomMoney - cutNumbers.lastObject.intValue + min)];;
    
    return [redPackets copy];
}
```

这种计算方法，能够保证红包的期望金额前后基本一致（除第一次外），而且最大额度可能会超过平均值的二倍。

但是，计算中多次进行了遍历和排序，效率较差，而且，红包金额是一次性算好返回，不能动态实时计算。

```objective-c
// 2000 的总额度，分配给 20 个单位，重复执行 100万 次，对应 index 元素相加的结果
(
    99601655,
    100176798,
    99913166,
    99947888,
    100075829,
    100120743,
    100069543,
    100100767,
    99850828,
    100060333,
    99873229,
    100055352,
    99899778,
    99874568,
    99880562,
    99911916,
    100177540,
    99936865,
    100028315,
    100444325
)
// 总用时 13.38369344297098
```

