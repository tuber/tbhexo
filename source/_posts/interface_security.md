---
title: 接口调用安全性
date: 2016-12-05 16:42:23
categories: PHP
tags:
 - 接口调用
 - 安全性
 - sign&token
---

一些接口调用中，为了安全原因，会加入一些token或者是签名。
以下是个简单例子：根据post给被调用方的key和value数组，
其中先根据数组做ksort，然后对value的值做（递归）拼接得到$string。
<!-- more -->
返回后用key.$string，调用方和被调用方按照此规则，对传递过来的额外post字段中的sign做验证，另外也可以结合一些非对称加密和对称加密，交换固定的字段和值来生成token，（此token也是固定的，可以随着时间等变化来变化），这样达到一定的安全性。

```php
function getSign($data, $key = '')
{
    if(is_array($data)){
        ksort($data);//按照键值排序
    }

    $string = getPostString($data);
    return $string;
    // return md5($string.$key);
}
/**
 * 数组系列化成字符串
 */
function getPostString(&$post)
{
    $string = '';
    if(is_array($post))
    {
        foreach($post as $item)
        {
            if(is_array($item))
                $string .= getPostString($item);
            else
                $string .= $item;
        }
    }
    else
    {
        $string = $post;
    }

    return $string;
}
$params=[
'mobile'=>'18618824588',
'code'=>'114360',
'hid'=>'2700',
'name'=>'tb',
'city_serial_id'=>'bj2700',
];

var_dump(ksort($params));
var_dump($params);
echo getSign($params);
```
