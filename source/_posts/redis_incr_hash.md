---
title: redis竟态与争抢唯一
date: 2017-11-22 21:12:03
categories: REDIS
tags:
- REDIS应用
---



## [incr][1]

比如北京车牌采取先抢到后审批资质的流程。车牌池子中有N多号码，页面呈现以一页十条的方式展示，每个号码后有一个抢的按钮，且一个人只能抢一个车牌，同样一个车牌只能被一个人抢到。[业务模型参考][2]

<!-- more -->
```
 if ($this->redis_db->incr("bj_".$car_no) != 1) {
      让别人先下手了,点别的去~
  }else{
       //抢到竞态条件，如果不复核资质要求退出，并清除incr
       if(抢到了但是没资质等){
         释放对此id的竟态权，别占茅坑
         $this->yredis_db->del("bj_".$id);
       }else{
         其他业务A
         抱得号码归...
         其他业务B
     }
}
```
另外，`incr`对`string`类型，`hash`类型，`sortedSet`类型都可以进行操作

## [blpop][3]

`blpop`相对于lpop有一个好处，可以对多个队列进行优先级操作。
`blpop`会按照`key`的排列顺序依次弹出，返回值为`key`的listname及具体元素值，而且可以设置`block`时间，原则是先阻塞先服务.

```
        $date = date('Ymd', time());
        //左进左出 ，优先分配一般的车牌号码，然后在分配非常好的连号号码，设置一个阻塞时间
        return $this->redis->blpop(self::$_config['dispatch_normal_list'] . $date, self::$_config['dispatch_better_list'] . $date, self::$_config['redis_block_l_pop_time_out']);

```

## [hsetnx][4]

设置`hash`中一个`field`为指定`value`，前提是`field`不存在。如果存在，返回0。
这样能保证在一个人只能抢一个车牌，但是抢到车牌执行付款或者其他业务操作过程中，其他人无法对此操作，（即不能将此车牌绑定到其他人身上）。根据具体业务情况，可设置基于car_no的hash field和基于 people 的hash field。

```
hash_base_people {"zhangsan":"京A888","lisi":"京A999"}
hash_base_car_no {"京A888":"zhangsan","京A999":"lisi"}
```

基于这两个hash 可以做更多关于业务的操作,比如通过hget等查看具体的绑定关系。

## [hdel][5]

有了通过hsetnx的绑定模型，当某个人对某个车牌交付了订金等一系列之后，就代表可以永远的将其消除，这样会用到hdel。另外如果在指定时间内没有做比如交付订金之类的操作，这个车牌号码会回炉到原始列表中。

```
 //删除以people_id为key的hash
 $base_people_id_del = $this->redis->hdel(self::$_config['hash_base_people'], $people_id);

 //删除以car_no为key的hash
 $base_car_no_del = $this->redis->hdel(self::$_config['hash_base_car_no'], $clue_id);

```




  [1]: http://redisdoc.com/string/incr.html
  [2]: http://num.10010.com/NumApp/chseNumList/init?num=186
  [3]: http://redisdoc.com/list/blpop.html
  [4]: http://redisdoc.com/hash/hsetnx.html
  [5]: http://redisdoc.com/hash/hdel.html
