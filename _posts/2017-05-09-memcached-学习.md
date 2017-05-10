# memcached 学习
## 一、Memcached内存分配
### 1. slab 划分内存空间
memcached 进程开启后，会划分一段连续的内存空间（默认64mb），之后再将内存空间划分成不同的slab。每一个slab只负责一定范围内的数据。只有满足该大小范围的数据才会存储到对应的slab中。
![](../../../images/pic1.png)

如上图，1-88byte 大小的数据将存储至slab1中。89-112bytes大小的数据将存放在slab2中。memcached 默认情况下下一个slab中的chunk大小是前一个的1.25倍。该增长因子可以在启动memcached 时设定。（参数 -f）

### 2. slab内存分配方式
memcached 在启动时可以使用 –m 制定最大使用内存，但是不会一启动就全部占用，而是随需要逐步分配给slab的。当向slab存储数据时，slab首先会申请一个page（默认1mb），再将此page分配成大小相同的chunk。memcached 不会释放已经分配的内存。Memcached分配出去的page不会被回收或者重新分配。Slab空闲的chunk不会借给其他slab使用。

## 3. 缓存释放策略
memcached 不会释放已经分配的内存。当已经分配内存所在位置的数据过期后，客户端无法再看见该记录。memcached不会监视记录是否过期，而是在get时查看记录的时间戳，检查记录是否过期。缓存清空策略：LRU。当内存空间不足时memcached 会优先使用已经超时的的记录的空间，如果还是存在新增数据空间不足时则会删除最近最少使用的记录。

## 二、memcached常用命令
### 1. 存储类命令
```
set       #设置一个键；
add       #添加一个新键；
replace   #替换一个现有键的值；
append    #在一个以存在键的值后面新增一个值；
prepend   #在一个以存在键的前面新增一个值；
```
共用的语法是：
```
>>> command <key_name> <flags> < timeout> <datasize> 
>>> <value>
```
```
key_name – key用于查找缓存值。
flags    – 可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息。
timeout  – 在缓存中保存键值对的时间长度（以秒为单位，0表示永远）。
datasize – 在缓存中存储的字节点，以字节为单位。
value    – 存储的值（始终位于第二行）。
```

### 2. 获取数据类命令
```
get       #或取一个键的值；
delete    #删除一个键；
incr      #让某些键自动加1；
decr      #让某些键自动减1；
```
共用的语法是：
```
>>> command key_name
```

### 3. 统计类命令
1）stats - 查看memcached 状态
```
stats
STAT pid 6624                  #Memcache服务器的进程ID
STAT uptime 557                #服务器已经运行的秒数
STAT time 1453224110           #服务器当前的UNIX时间戳
STAT version 1.4.25            #Memcache的版本号
STAT libevent 2.0.22-stable    #Libevent的版本号（memcached使用libevent epoll模型）
STAT pointer_size 64           #当前操作系统的指针大小
STAT rusage_user 0.016997      #进程的累计用户时间
STAT rusage_system 0.049992    #进程的累计系统时间
STAT curr_connections 10       #当前打开着的连接数
STAT total_connections 13      #从服务器启动以后曾经打开过的连接数
STAT connection_structures 11  #服务器分配的连接构造数
STAT reserved_fds 20           #内部使用的FD数
STAT cmd_get 2                 #get命令获取总请求数
STAT cmd_set 5                 #set命令获取总请求数
STAT cmd_flush 0               #flush命令获取总请求数
STAT cmd_touch 0               #touch命令获取总请求数
STAT get_hits 1                #get命令命中次数
STAT get_misses 1              #get命令未命中次数
STAT delete_misses 0           #delete命令未命中次数
STAT delete_hits 0             #delete命令命中次数
STAT incr_misses 0             #incr命令未命中次数
STAT incr_hits 0               #incr命令命中次数
STAT decr_misses 0             #decr命令未命中次数
STAT decr_hits 0               #decr命令命中次数
STAT cas_misses 0              #cas命令未命中次数
STAT cas_hits 0                #cas命令命中次数
STAT cas_badval 0              #使用擦拭次数
STAT touch_hits 0              #touch命令命中次数
STAT touch_misses 0            #touch命令未命中次数
STAT auth_cmds 0               #认证命令处理的次数
STAT auth_errors 0             #认证失败数目
STAT bytes_read 244            #总读取字节数
STAT bytes_written 173         #总发送字节数
STAT limit_maxbytes 67108864   #分配给memcache的内存大小
STAT accepting_conns 1         #接受新的连接
STAT listen_disabled_num 0     #失效的监听数
STAT time_in_listen_disabled_us 0
STAT threads 4                 #当前线程数
STAT conn_yields 0             #连接操作主动放弃数目
STAT hash_power_level 16       #hash表等级
STAT hash_bytes 524288         #当前hash表大小
STAT hash_is_expanding 0       #hash表正在扩展
STAT malloc_fails 0
STAT bytes 76                  #当前服务器存储items占用的字节数
STAT curr_items 1              #服务器当前存储的item数量
STAT total_items 2             #从服务器启动以后存储的items总数量
STAT expired_unfetched 0       #已过期但未获取的对象数目
STAT evicted_unfetched 0       #已驱逐但未获取的对象数目
STAT evictions 0               #LRU释放的对象数目
STAT reclaimed 0               #已过期的数据条目来存储新数据的数目
STAT crawler_reclaimed 0
STAT crawler_items_checked 0
STAT lrutail_reflocked 0
```

2) stats items – 查看item信息，每一组slab对应一组item值，一下显示一组items

``` 
stats items
STAT items:1:number 18184
STAT items:1:age 2207
STAT items:1:evicted 0
STAT items:1:evicted_nonzero 0
STAT items:1:evicted_time 0
STAT items:1:outofmemory 0
STAT items:1:tailrepairs 0
STAT items:1:reclaimed 0
STAT items:1:expired_unfetched 0
STAT items:1:evicted_unfetched 0
STAT items:1:crawler_reclaimed 0
STAT items:1:lrutail_reflocked 0

```

3) stats slabs – 查看slab信息，这里只显示一组slab信息，默认有多少slab就会有多少信息
```
stats slabs
STAT 1:chunk_size 96          #此组slabs中chunk的大小；
STAT 1:chunks_per_page 10922
STAT 1:total_pages 2          #此组slabs当前一共申请了多少个Page（一个Page为1M）；
STAT 1:total_chunks 21844     #此组slabs一共有多少个chunk；
STAT 1:used_chunks 18498      #此组slabs当前正在使用的chunk数量；
STAT 1:free_chunks 3346       #词组slabs当前未使用的chunk数量；
STAT 1:free_chunks_end 0
STAT 1:mem_requested 1704171
STAT 1:get_hits 695746
STAT 1:cmd_set 160754
STAT 1:delete_hits 14339
STAT 1:incr_hits 0
STAT 1:decr_hits 0
STAT 1:cas_hits 0
STAT 1:cas_badval 0
STAT 1:touch_hits 0
STAT active_slabs 1           #此memcached实例一共有多少个slabs；
STAT total_malloced 1048512
```
通过查看slabs信息，可以确定每个slabs占用了多少内存，一个有多少个chunk，每个chunk有多大，以及当前未使用的chunk数量。

4) stats sizes – 查看key存储大小以及对应的数量，可以统计出实际存储大小；
```
stats sizes
STAT 96 18624
...
```

5) 清除命令

	flush_all


6) 退出命令
	quit

## 三、如何计算内存浪费
使用 stats slab 查看所有slab使用情况。
使用 ps或者top 查看memcached使用内存情况。
使用stats sizes 查看实际存储大小。

由于memcached不会主动释放已经分配的page。所以会存在一种情况：一个slab曾经申请了大量page但是数据过期后不再存储如此大量的数据，从而造成大量page浪费。其他slab在内存使用完毕后申请不到新page于是触发了LRU机制，删除了一些有效的数据。解决方案：重启memcached服务。

检查各个slab中数据存储的情况，检查是否在某些特定的trunk中存在内存浪费（比较大的chunk中存入大量稍大于最小容量的数据，或者存在大量小slab空间未使用）。适度调整slab中trunk大小递增的倍率。（参数 -f）

## 四、测试
```
<?php
$mem = new Memcached();
$mem->addServer('localhost', 11211);
//$mem->addServer('localhost', 11212);

for($i = 0; $i < 5000; $i++) {
    $keyName = 'key:' . $i;
    $mem->get($keyName);

    $msg = createRandomMsg(rand(1, 9999));
    $mem->set($keyName, $msg, 86400);
}

//echo createRandomMsg();

/**
 * create a random message with given length
 *
 * @param int $length
 * @return string
 */
function createRandomMsg($length = 1) {
    $msg = '';
    $allKey = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnoprstuvwxyz1234567890';
    for ($i = 0; $i < $length; $i++) {
        $msgKey = rand(0, strlen($allKey)-1);
        $msg .= $allKey[$msgKey];
    }

    return $msg;
}
```
向memcached中随机存入1-9999byte的数据5000个，过期时间设置为1天。

![](../../../images/pic2.png)
总共使用了 43.9MB 其中浪费了19.5MB的内存

![](../../../images/pic3.png)
查看了slab使用情况 小容量的slab使用率很低。于是我重启服务将容量递增倍率调整为 1.5 重新写入5000个数据

![](../../../images/pic4.png)

可以看见浪费的内存减少了。

