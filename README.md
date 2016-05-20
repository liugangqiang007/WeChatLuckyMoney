# WeChatLuckyMoney
微信（WeChat）随机红包金额分配算法

```
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