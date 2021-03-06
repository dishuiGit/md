[笔记列表][anchor-id]

[anchor-id]: file:///D:/1125/%E7%AC%94%E8%AE%B0/note/html/
[anchor-cur]: #
[TOC]

## [ 一、概述][anchor-cur]

> + Sorted-Sets和Sets类型极为相似，它们都是字符串的集合，都不允许重复的成员出现在一个Set中。
+ 它们之间的主要差别是Sorted-Sets中的每一个成员都会有一个分数(score)与之关联，
+ Redis正是通过分数来为集合中的成员进行从小到大的排序。
+ 然而需要额外指出的是，尽管Sorted-Sets中的成员必须是唯一的，但是分数(score)却是可以重复的。
+ 在Sorted-Set中添加、删除或更新一个成员都是非常快速的操作，其时间复杂度为集合中成员
    数量的对数。
+ 由于Sorted-Sets中的成员在集合中的位置是有序的，因此，即便是访问位于集合中部的成员
    也仍然是非常高效的。
+ 事实上，Redis所具有的这一特征在很多其它类型的数据库中是很难实现的，
    换句话说，在该点上要想达到和Redis同样的高效，在其它数据库中进行建模是非常困难的。

## [ 二、相关命令列表][anchor-cur]

+ `ZADD key score member [score] [member]`   
    + 添加参数中指定的所有成员及其分数到指定key的Sorted-Set中，
        在该命令中我们可以指定多组score/member作为参数。
    + 如果在添加时参数中的某一成员已经存在，该命令将更新此成员的分数为新值，同时再将该成
        员基于新值重新排序。
    + 如果键不存在，该命令将为该键创建一个新的Sorted-Sets Value，并将score/member对插入其中。
    + 如果该键已经存在，但是与其关联的Value不是Sorted-Sets类型，相关的错误信息将被返回。    
        本次操作实际插入的成员数量。
+ `ZCARD key`    
    + 获取与该Key相关联的Sorted-Sets中包含的成员数量。    
+ 返回Sorted-Sets中的成员数量，如果该Key不存在，返回0。

+ `ZCOUNT key min max `  
    + 该命令用于获取分数(score)在min和max之间的成员数量。针对min和max参数需要额外说明的是，
         -inf和+inf分别表示Sorted-Sets中分数的最高值和最低值。
    + 缺省情况下，min和max表示的范围是闭区间范围，即min <= score <= max内的成员将被返回。
    + 然而我们可以通过在min和max的前面添加"("字符来表示开区间，如(min max表示min < score <=  
        max，而(min (max表示min < score < max。    分数指定范围内成员的数量。
+ `ZINCRBY key increment member`     
    + 该命令将为指定Key中的指定成员增加指定的分数。
        + 如果成员不存在，该命令将添加该成员并假设其初始分数为0，此后再将其分数加上increment。
        + 如果Key不存，该命令将创建该Key及其关联的Sorted-Sets，并包含参数指定的成员，
            其分数为increment参数。
    + 如果与该Key关联的不是Sorted-Sets类型，相关的错误信息将被返回。    
+ 以字符串形式表示的新分数。
+ `ZRANGE key start stop [WITHSCORES]`   
    + 该命令返回顺序在参数start和stop指定范围内的成员，
        这里start和stop参数都是0-based，即0表示第一个成员，-1表示最后一个成员。
        + 如果start大于该Sorted-Set中的最大索引值，或start > stop，此时一个空集合将被返回。
        + 如果stop大于最大索引值，该命令将返回从start到集合的最后一个成员。
        + 如果命令中带有可选参数WITHSCORES选项，该命令在返回的结果中将包
            含每个成员的分数值，如value1,score1,value2,score2...。　　  
+ 返回索引在start和stop之间的成员列表。

+ `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`      
    + 该命令将返回分数在min和max之间的所有成员，即满足表达式min <= score <= 
        max的成员，其中返回的成员是按照其分数从低到高的顺序返回，
    + 如果成员具有相同的分数，则按成员的字典顺序返回。可选参数LIMIT用于限制返回成员的数量范围    。可选参数offset表示从符合条件的第offset个成员开始返回，同时返回count个成员。
    + 可选参数WITHSCORES的含义参照ZRANGE中该选项的说明。
    + 最后需要说明的是参数中min和max的规则可参照命令ZCOUNT。   
        返回分数在指定范围内的成员列表。
+ `ZRANK key member`     
    + Sorted-Set中的成员都是按照分数从低到高的顺序存储，该命令将返回参数中指定
        成员的位置值，其中0表示第一个成员，它是Sorted-Set中分数最低的成员。   
    + 如果该成员存在，则返回它的位置索引值。否则返回nil。
+ `ZREM key member [member ...]`     
    + 该命令将移除参数中指定的成员，其中不存在的成员将被忽略。
        + 如果与该Key关联的Value不是Sorted-Set，相应的错误信息将被返回。      
            实际被删除的成员数量。
+ `ZREVRANGE key start stop [WITHSCORES]`    
    + 该命令的功能和ZRANGE基本相同，唯一的差别在于该命令是通过反向排序获取指定位置的成员，即从
        高到低的顺序。
    + 如果成员具有相同的分数，则按降序字典顺序排序。返回指定的成员列表。
+ `ZREVRANK key member`      
    + 该命令的功能和ZRANK基本相同，唯一的差别在于该命令获取的索引是从高到低
        排序后的位置，同样0表示第一个元素，即分数最高的成员。   
    + 如果该成员存在，则返回它的位置索引值。否则返回nil。 

+ `ZSCORE key member`    
    + 获取指定Key的指定成员的分数。   
    + 如果该成员存在，以字符串的形式返回其分数，否则返回nil。
+ `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]`   
    + 该命令除了排序方式是基于从高到低的分数排序之外，其它功能和参数含义均与ZRANGEBYSCORE相同。    
    + 返回分数在指定范围内的成员列表。 

+ `ZREMRANGEBYRANK key start stop`   
    + 删除索引位置位于start和stop之间的成员，start和stop都是0-based，
        即0表示分数最低的成员，-1表示最后一个成员，即分数最高的成员。   
    + 被删除的成员数量。
+ `ZREMRANGEBYSCORE key min max `    
    + 删除分数在min和max之间的所有成员，即满足表达式min <= score <= max的所有成员。
    + 对于min和max参数，可以采用开区间的方式表示，具体规则参照ZCOUNT。  
        被删除的成员数量。

## [ 三、命令示例：][anchor-cur]

### 1. ZADD/ZCARD/ZCOUNT/ZREM/ZINCRBY/ZSCORE/ZRANGE/ZRANK:
+ `/> redis-cli`
    + 在Shell的命令行下启动Redis客户端工具。
    + + `redis 127.0.0.1:6379> zadd myzset 1 "one"`
    + 添加一个分数为1的成员。
    + (integer) 1
+ `redis 127.0.0.1:6379> zadd myzset 2 "two" 3 "three"`
    + 添加两个分数分别是2和3的两个成员。
    + (integer) 2
+ `redis 127.0.0.1:6379> zrange myzset   0 -1 WITHSCORES`
    + 0表示第一个成员，-+ 表示最后一个成员。
    + W + ITHSCORES选项表示返回的结果中包含每个成员及其分数，否则只返回成员。
        + 1) "one"
            2) "1"
            3) "two"
            4) "2"
            5) "three"
            6) "3"
+ `redis 127.0.0.1:6379> zrank myzset one`
    + 获取成员one在Sorted-Set中的位置索引值。0表示第一个位置。
    + (integer) 0
+ `redis 127.0.0.1:6379> zrank myzset four`
    + 成员four并不存在，因此返回nil。
    + (nil)
+ `redis 127.0.0.1:6379> zcard myzset`
    + 获取myzset键中成员的数量。    
    + (integer) 3
+ `redis 127.0.0.1:6379> zcount myzset 1 2`
    + 返回与myzset关联的Sorted-Set中，分数满足表达式1 <= score <= 2的成员的数量。
    + (integer) 2
+ `redis 127.0.0.1:6379> zrem myzset one two`
    + 删除成员one和two，返回实际删除成员的数量。
    + (integer) 2
+ `redis 127.0.0.1:6379> zcard myzset`
    + 查看是否删除成功。
    + (integer) 1
+ `redis 127.0.0.1:6379> zscore myzset three`
    + 获取成员three的分数。返回值是字符串形式。
    + "3"
+ `redis 127.0.0.1:6379> zscore myzset two`
    + 由于成员two已经被删除，所以该命令返回nil。
    + (nil)
+ `redis 127.0.0.1:6379> zincrby myzset 2 one`
    + 将成员one的分数增加2，并返回该成员更新后的分数。
    + "3"
+ `redis 127.0.0.1:6379> zincrby myzset -1 one`
    + 将成员one的分数增加-1，并返回该成员更新后的分数。
    + "2"
+ `redis 127.0.0.1:6379> zrange myzset 0 -1 WITHSCORES`
    + 查看在更新了成员的分数后是否正确。
    + 1) "one"
        2) "2"
        3) "two"
        4) "2"
        5) "three"
        6) "3"

### 2. ZRANGEBYSCORE/ZREMRANGEBYRANK/ZREMRANGEBYSCORE
+ `redis 127.0.0.1:6379> del myzset`
    + (integer) 1
+ `redis 127.0.0.1:6379> zadd myzset 1 one 2 two 3 three 4 four`
    + (integer) 4
+ `redis 127.0.0.1:6379> zrangebyscore myzset 1 2`
    + 获取分数满足表达式1 <= score <= 2的成员。
    + 1) "one"
        2) "two"
+ `redis 127.0.0.1:6379> zrangebyscore myzset (1 2`
    + 获取分数满足表达式1 < score <= 2的成员。
    + 1) "two"
+ `redis 127.0.0.1:6379> zrangebyscore myzset -inf +inf limit 2 3`
    + inf表示第一个成员，+inf表示最后一个成员，limit后面的参数用于限制返回成员的自己，
        2表示从位置索引(0-based)等于2的成员开始，去后面3个成员。
        + 1) "three"
            2) "four"
+ `redis 127.0.0.1:6379> zremrangebyscore myzset 1 2`
    + 删除分数满足表达式1 <= score <= 2的成员，并返回实际删除的数量。
    + (integer) 2
+ `redis 127.0.0.1:6379> zrange myzset 0 -1`
    + 看出一下上面的删除是否成功。
    + 1) "three"
2) "four"
+ `redis 127.0.0.1:6379> zremrangebyrank myzset 0 1`
    + 删除位置索引满足表达式0 <= rank <= 1的成员。
    + (integer) 2
+ `redis 127.0.0.1:6379> zcard myzset`
    + 查看上一条命令是否删除成功。
    + (integer) 0
  
### 3. ZREVRANGE/ZREVRANGEBYSCORE/ZREVRANK:
+ `redis 127.0.0.1:6379> del myzset`
    + 为后面的示例准备测试数据。
    + (integer) 0
+ `redis 127.0.0.1:6379> zadd myzset 1 one 2 two 3 three 4 four`
    + (integer) 4
+ `redis 127.0.0.1:6379> zrevrange myzset 0 -1 WITHSCORES`
    + 以位置索引从高到低的方式获取并返回此区间内的成员。
    + 1) "four"
        2) "4"
        3) "three"
        4) "3"
        5) "two"
        6) "2"
        7) "one"
        8) "1"
+ `redis 127.0.0.1:6379> zrevrange myzset 1 3`
    + 由于是从高到低的排序，所以位置等于0的是four，1是three，并以此类推。
    + 1) "three"
        2) "two"
        3) "one"
+ `redis 127.0.0.1:6379> zrevrank myzset one`
    + 由于是从高到低的排序，所以one的位置是3。
    + (integer) 3
+ `redis 127.0.0.1:6379> zrevrank myzset four`
    + 由于是从高到低的排序，所以four的位置是0。
    + (integer) 0
+ `redis 127.0.0.1:6379> zrevrangebyscore myzset 3 0`
    + 获取分数满足表达式3 >= score >= 0的成员，并以相反的顺序输出，即从高到底的顺序。
    + 1) "three"
        2) "two"
        3) "one"
+ `redis 127.0.0.1:6379> zrevrangebyscore myzset 4 0 limit 1 2`
    + 该命令支持limit选项，其含义等同于zrangebyscore中的该选项，只是在计算位置时按照相反的顺序
        计算和获取。
    + 1) "three"
        2) "two"
    
## [ 四、应用范围：][anchor-cur]

```text
1). 可以用于一个大型在线游戏的积分排行榜。每当玩家的分数发生变化时，可以执行ZADD命令更新玩家的
分数，此后再通过ZRANGE命令获取积分TOPTEN的用户信息。当然我们也可以利用ZRANK命令通过username来
获取玩家的排行信息。最后我们将组合使用ZRANGE和ZRANK命令快速的获取和某个玩家积分相近的其他用户
的信息。
2). Sorted-Sets类型还可用于构建索引数据。
```