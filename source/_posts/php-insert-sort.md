---
title: PHP的插入排序
date: 2016-11-04 19:18:59
categories: PHP
tags:
 - PHP
 - 插入排序
 - 算法
---
```php
<?php
/*
1.外层循环是从数组中选出一个arr[i]将要插入到有序数组的数
2.内层循环是遍历已经排序好的数组，将arr[i]（也就是temp）依次与有序数组做对比，如果发现有序数组其中一个比准备插进来的arr[i]大，
那么谁比这个arr[i]大，谁就出去，把位置腾出去，当然比arr[i]大的这个数也不能扔，就给他放在已经排号序数组中且相对于
他的下一个索引就好了。
3.2还有个空档，这时候把temp补上就行了
*/
$arr = [3, 7, 6,8,1];

function insertSort($arr)
{
    for ($i = 1; $i < count($arr); $i++)
    {
        echo '外层{$i}=' . $i . "次循环" . "\n";

        $temp = $arr[$i];//待插入的数
        echo '外层待插入的数为{$temp}=' . $temp . "\n";

        for ($j = $i - 1; $j >= 0 && $temp > $arr[$j]; $j--)
        {
            echo '====>内层{$j}=' . $j . "次循环" . "\n";
            echo '要插入的$temp' . "=$temp " . "VS " . '已经排序好的$arr[$j]=' . "$arr[$j]" . "小\n";

            $arr[$j + 1] = $arr[$j];
            echo '所以要腾出一个空来给temp,这时候索引$j的值为'.$arr[$j].'要往后靠，此时$j=' . $j, "\n";
        }

        echo '外层{$i}=' . $i . '次循环结束';
        // var_dump($temp,$arr[$i]);注意此处两值如果已经经历了内层循环，那么就！==
        $arr[$j + 1] = $temp;
        echo '  此时完成$j=' . $j . '把temp的值' . "$temp" . '赋予$j+1' . "\n";

        echo "=====", "\n";

    }
    return $arr;
}

var_dump(insertSort($arr));
```
