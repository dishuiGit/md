[笔记列表][anchor-id]

[anchor-id]: file:///D:/1125/%E7%AC%94%E8%AE%B0/note/html/
[anchor-cur]: #
[TOC]

## [ 一、概述][anchor-cur]

>   在Redis中，我们可以将Set类型看作为没有排序的字符集合，和List类型一样，
我们也可以在该类型的数据值上执行添加、删除或判断某一元素是否存在等操作。
需要说明的是，这些操作的时间复杂度为O(1)，即常量时间内完成次操作。

>   Set可包含的最大元素数量是4294967295。
和List类型不同的是，Set集合中不允许出现重复的元素，
这一点和C++标准库中的set容器是完全相同的。换句话说，如果多次添加相同元素，
Set中将仅保留该元素的一份拷贝。
和List类型相比，Set类型在功能上还存在着一个非常重要的特性，即在服务器端
完成多个Sets之间的聚合计算操作，如unions、intersections和differences。
由于这些操作均在服务端完成，因此效率极高，而且也节省了大量的网络IO开销。


## [ 二、相关命令列表：][anchor-cur]
+ SADD key member [member ...]     
    + 如果在插入的过程用，参数中有的成员在Set中已经存在，该成员将被忽略，
        而其它成员仍将会被正常插入。
    + 如果执行该命令之前，该Key并不存在，该命令将会创建一个新的Set，
        此后再将参数中的成员陆续插入。
    + 如果该Key的Value不是Set类型，该命令将返回相关的错误信息。   
        本次操作实际插入的成员数量。

+ SCARD key    
    + 获取Set中成员的数量。   
    + 返回Set中成员的数量，如果该Key并不存在，返回0。

+ SISMEMBER key member     
    + 判断参数中指定成员是否已经存在于与Key相关联的Set集合中。    
    + 1表示已经存在，0表示不存在，或该Key本身并不存在。

+ SMEMBERS key     
    + 获取与该Key关联的Set中所有的成员。
    + 返回Set中所有的成员。

+ Skey     
    + 随机的移除并返回Set中的某一成员。 
    + 由于Set中元素的布局不受外部控制，因此无法像List那样确定哪个元素位于Set的
        头部或者尾部。    
+ 返回移除的成员，如果该Key并不存在，则返回nil。

+ SREM key member [member ...]     
    + 从与Key关联的Set中删除参数中指定的成员，不存在的参数成员将被忽略，
    + 如果该Key并不存在，将视为空Set处理。  
+ 从Set中实际移除的成员数量，如果没有则返回0。

+ SRANDMEMBER key      
    + 和SPOP一样，随机的返回Set中的一个成员，
        不同的是该命令并不会删除返回的成员。  
    + 返回随机位置的成员，如果Key不存在则返回nil。

+ SMOVE source destination member  
    + 原子性的将参数中的成员从source键移入到destination键所关联的Set中。
    + 因此在某一时刻，该成员或者出现在source中，或者出现在destination中。
    + 如果该成员在source中并不存在，该命令将不会再执行任何操作并返回0，否
        则，该成员将从source移入到destination。
    + 如果此时该成员已经在destination中存在，那么该命令仅是将该成员从source中移出。
    + 如果和Key关联的Value不是Set，将返回相关的错误信息。   
        + 1表示正常移动，0表示source中并不包含参数成员。

+ SDIFF key [key ...]  
    + 返回参数中第一个Key所关联的Set和其后所有Keys所关联的Sets中成员的差异。
    + 如果Key不存在，则视为空Set。    差异结果成员的集合。

    + SDIFFSTORE destination key [key ...]     
    + 该命令和SDIFF命令在功能上完全相同，两者之间唯一的差别是SDIFF返回差异的结果成
        员，而该命令将差异成员存储在destination关联的Set中。
    + 如果destination键已经存在，该操作将覆盖它的成员。 
+ 返回差异成员的数量。
+ SINTER key [key ...]     
    + 该命令将返回参数中所有Keys关联的Sets中成员的交集。
    + 因此如果参数中任何一个Key关联的Set为空，或某一Key不存在，那么该命令的结果
        将为空集。    交集结果成员的集合。
+ SINTERSTORE destination key [key ...]    
    + 该命令和SINTER命令在功能上完全相同，两者之间唯一的差别是SINTER返回交集的结果
        成员，而该命令将交集成员存储在destination关联的Set中。
    + 如果destination键已经存在，该操作将覆盖它的成员。   
+ 返回交集成员的数量。 

+ SUNION key [key ...]     
    + 该命令将返回参数中所有Keys关联的Sets中成员的并集。  并集结果成员的集合。

+ SUNIONSTORE destination key [key ...]    
    + 该命令和SUNION命令在功能上完全相同，两者之间唯一的差别是SUNION返回并集的
        结果成员，而该命令将并集成员存储在destination关联的Set中。
    + 如果destination键已经存在，该操作将覆盖它的成员。   
+ 返回并集成员的数量。

## [ 三、命令示例][anchor-cur]

### 1. SADD/SMEMBERS/SCARD/SISMEMBER
+ `/> redis-cli`
    + 在Shell命令行下启动Redis的客户端程序。
+ `redis 127.0.0.1:6379> sadd myset a b c`
    + 插入测试数据，由于该键myset之前并不存在，因此参数中的三个成员都被正常插入。
    + (integer) 3
+ `redis 127.0.0.1:6379> sadd myset a d e`
    + 由于参数中的a在myset中已经存在，因此本次操作仅仅插入了d和e两个新成员。
    + (integer) 2
+ `redis 127.0.0.1:6379> sismember myset a`
    + 判断a是否已经存在，返回值为1表示存在。
    + (integer) 1
+ `redis 127.0.0.1:6379> sismember myset f`
    + 判断f是否已经存在，返回值为0表示不存在。
    + (integer) 0
+ `redis 127.0.0.1:6379> smembers myset`
    + 通过smembers命令查看插入的结果，从结果可以，输出的顺序和插入顺序无关。
    + 1) "c"
        2) "d"
        3) "a"
        4) "b"
        5) "e"
+ `redis 127.0.0.1:6379> scard myset`
    + 获取Set集合中元素的数量。
    + (integer) 5

### 2. SPOP/SREM/SRANDMEMBER/SMOVE:
+ `redis 127.0.0.1:6379> del myset`
    + 删除该键，便于后面的测试。
    + (integer) 1
+ `redis 127.0.0.1:6379> sadd myset a b c d`
    + 为后面的示例准备测试数据。
    + (integer) 4
+ `redis 127.0.0.1:6379> smembers myset`
    + 查看Set中成员的位置。
    + 1) "c"
        2) "d"
        3) "a"
        4) "b"
+ `redis 127.0.0.1:6379> srandmember myset`
    + 从结果可以看出，该命令确实是随机的返回了某一成员。
    + "c"
+ `redis 127.0.0.1:6379> spop myset`
    + Set中尾部的成员b被移出并返回，事实上b并不是之前插入的第一个或最后一个成员。
    + "b"
+ `redis 127.0.0.1:6379> smembers myset`
    + 查看移出后Set的成员信息。
    + 1) "c"
        2) "d"
        3) "a"
+ `redis 127.0.0.1:6379> srem myset a d f`
    + 从Set中移出a、d和f三个成员，其中f并不存在，因此只有a和d两个成员被移出，返
        回为2  
    + (integer) 2
+ `redis 127.0.0.1:6379> smembers myset`
    + 查看移出后的输出结果。
    + 1) "c"
+ `redis 127.0.0.1:6379> sadd myset a b`
    + 为后面的smove命令准备数据。
    + (integer) 2
+ `redis 127.0.0.1:6379> sadd myset2 c d`
    + (integer) 2
+ `redis 127.0.0.1:6379> smove myset myset2 a`
    + 将a从myset移到myset2，从结果可以看出移动成功。
    + (integer) 1
+ `redis 127.0.0.1:6379> smove myset myset2 a`
    + 再次将a从myset移到myset2，由于此时a已经不是myset的成员了，因此移动失败并返回0。
    + (integer) 0
+ `redis 127.0.0.1:6379> smembers myset`
    + 分别查看myset和myset2的成员，确认移动是否真的成功。
    + 1) "b"
+ `redis 127.0.0.1:6379> smembers myset2`
    + 1) "c"
        2) "d"
        3) "a"

### 3. SDIFF/SDIFFSTORE/SINTER/SINTERSTORE:
+ `redis 127.0.0.1:6379> sadd myset a b c d`
    + 为后面的命令准备测试数据。
+ `redis 127.0.0.1:6379> sadd myset2 c`
    + (integer) 4
+ `redis 127.0.0.1:6379> sadd myset3 a c e`
    + (integer) 1
    + (integer) 3
+ `redis 127.0.0.1:6379> sdiff myset myset2 myset3`
    + myset和myset2相比，a、b和d三个成员是两者之间的差异成员。
        再用这个结果继续和myse
    + 3进行差异比较，b和d是myset3不存在的成员。
    + 1) "d"
        2) "b"
+ `redis 127.0.0.1:6379> sdiffstore diffkey myset myset2 myset3`
    + 将3个集合的差异成员存在在diffkey关联的Set中，并返回插入的成员数量。
    + (integer) 2
+ `redis 127.0.0.1:6379> smembers diffkey`
    + 查看一下sdiffstore的操作结果。
    + 1) "d"
        2) "b"
+ `redis 127.0.0.1:6379> sinter myset myset2 myset3`
    + 从之前准备的数据就可以看出，这三个Set的成员交集只有c。
    + 1) "c"
+ `redis 127.0.0.1:6379> sinterstore interkey myset myset2 myset3`
    + 将3个集合中的交集成员存储到与interkey关联的Set中，并返回交集成员的数量。
    + (integer) 1
+ `redis 127.0.0.1:6379> smembers interkey`
    + 查看一下sinterstore的操作结果。
    + 1) "c"
+ `redis 127.0.0.1:6379> sunion myset myset2 myset3`
    + 获取3个集合中的成员的并集。    
    + 1) "b"
        2) "c"
        3) "d"
        4) "e"
        5) "a"
+ `redis 127.0.0.1:6379> sunionstore unionkey myset myset2 myset3`
    + 将3个集合中成员的并集存储到unionkey关联的set中，并返回并集成员的数量。
    + (integer) 5
+ `redis 127.0.0.1:6379> smembers unionkey`
    + 查看一下suiionstore的操作结果。
    + 1) "b"
        2) "c"
        3) "d"
        4) "e"
        5) "a"

## [ 四、应用范围：][anchor-cur]

>1). 可以使用Redis的Set数据类型跟踪一些唯一性数据，比如访问某一博客的唯一IP地
址信息。对于此场景，我们仅需在每次访问该博客时将访问者的IP存入Redis中，Set数
据类型会自动保证IP地址的唯一性。
2). 充分利用Set类型的服务端聚合操作方便、高效的特性，可以用于维护数据对象之间
的关联关系。比如所有购买某一电子设备的客户ID被存储在一个指定的Set中，而购买另
外一种电子产品的客户ID被存储在另外一个Set中，如果此时我们想获取有哪些客户同时
购买了这两种商品时，Set的intersections命令就可以充分发挥它的方便和效率的优势
了。