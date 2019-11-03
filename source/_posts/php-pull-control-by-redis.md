---
title: php同步拉去大量数据的一种可控方法
date: 2019-07-20 13:18:59
categories: PHP
tags:
 - PHP
 - REDIS
---

## 场景
数据同步只能通过php脚本拉取三方接口来执行。比如我需要每天拉取从jd商城下单的数据到mysql，jd通过已知接口告知我共多少页多少条数据。大概每天60w条，但是问题是jd接口请求频次受限。而用php请求还有个问题就是脚本可能超时或者由于其他原因异常退出。这样会导致数据插入失败，甚至是插入重复。
<!-- more -->

## 实现思路
1. 通过接口查询当天总条数（假设在获取过程中数据变化可控）
2. 根据对方接口频次限制需求及自身机器性能，算出每页多少条可以查询（即插入）
3. mysql插入采用n条数据采用batch方式，缩短事务频次及语法解析频次等
4. 分段插入，插入成功后用redis设置offset，防止对方及自身进程异常退出
5. 在程序入口判断offset值，是否需要从断点处开始或者从1开始，或者是已经插入完成。

## php简单代码实现（ci框架）
```
<?php  
public function get_jd_order()
    {
        $is_success = false;
        $this->benchmark->mark('jd_order_begin');
        set_time_limit(0);
        ini_set('memory_limit', '2048M');
        //请求第一页获取总数
        $data = $this->_getOrderId(1);
        $totalPage = ceil($data['alltotal'] / 100);

        if ($this->yredis->get(self::_REQ_jd_PAGE_COMPLETE) <= 0) {
            $page = 1;
            $this->_log("初始化，从第一页开始");
        } else if ($page = $this->yredis->get(self::_REQ_jd_PAGE_INTERRUPT)) {
            $page = $this->yredis->get(self::_REQ_jd_PAGE_INTERRUPT);
            $this->_log("有中断，从第{$page}页再来");
        } else if ($this->yredis->get(self::_REQ_jd_PAGE_COMPLETE) == $totalPage) {
            $this->_log("{$totalPage} 页,总条数 {$data['alltotal']}都处理完毕，无需同步");
            exit;
        }
        if (!$totalPage) {
            exit('total page zero');
        } else {
            $this->_log("京东总页数:{$totalPage}，总条数{$data['alltotal']}");
        }

        for ($page; $page <= $totalPage; $page++) {
            //页数循环到20页时 休息2秒
//            if ($page % 20 == 0) {
//                sleep(2);
//            }
            $data = $this->_getOrderId($page);
            $tempOrderIdArr = $data['data'];
            $detailData = $this->_getOrderDetail($tempOrderIdArr);
            if (empty($detailData)) {
                //停两秒再次
                $this->_log("detail data empty，sleep 2 second ↓");
                sleep(2);
                $detailData = $this->_getOrderDetail($tempOrderIdArr);
                if (empty($detailData)) {
                    $this->_log("again detail data empty，exit ↓");
                    $this->_log(implode(',', $tempOrderIdArr));
                    $this->yredis->set(self::_REQ_jd_PAGE_INTERRUPT, $page);
                    $this->_log('$detailData为空');
                    exit();
                }
            }
            $insert_data = [];
            foreach ($detailData as $key => $value) {
                $insert_data[] = array_merge($value, [
                    'order_id' => $key,
                    'from_source' => '3', //订单来源: 京东
                    'data_generate_date' => date('Y-m-d', time()),
                ]);
            }
            echo "  {$page}:获取京东订单第{$page}页\r\n";
            echo "  {$page}:准备插入: " . count($insert_data) . "条" . "\r\n";
//            var_dump(array_diff(array_keys($insert_data['86']),array_keys($insert_data['87'])));die;
            $this->crm_w->trans_begin();
            $this->crm_w->insert_batch($this->_insert_table, $insert_data);
            if ($this->crm_w->affected_rows() < 0) {
                echo "插入失败,失败详情↓\r\n";
                $is_success = false;
            } else {
                echo "插入完成(ci2.0 insert batch 方法100问题): " . $this->crm_w->affected_rows() . "\r\n";
            }
            if ($this->crm_w->trans_status() === FALSE) {
                var_dump($this->crm_w->last_query());
                $this->_log("在第{$page}页有错误 ");
                $this->yredis->set(self::_REQ_jd_PAGE_INTERRUPT, $page);
                $this->_log("需要从{$page}页重新开始 ");
                var_dump($this->crm_w->_error_message());
                $is_success = false;
                exit;
            } else {
                $this->crm_w->trans_complete();
                $this->_log("第{$page}页 存储Mysql成功");
                $this->yredis->set(self::_REQ_jd_PAGE_COMPLETE, $page);
                if ($this->yredis->get(self::_REQ_jd_PAGE_COMPLETE) > $this->yredis->get(self::_REQ_jd_PAGE_INTERRUPT)) {
                    $this->yredis->del(self::_REQ_jd_PAGE_INTERRUPT);
                }
                $this->_log("第{$page}页 Redis Set成功");
            }
        }

        $this->yredis->set_timeout(self::_REQ_jd_PAGE_COMPLETE, self::_REQ_jd_PAGE_COUNT_TTL);
        $this->yredis->set_timeout(self::_REQ_jd_PAGE_INTERRUPT, self::_REQ_jd_PAGE_COUNT_TTL);
        $this->_log("京东拉取完成,总时长为 " . $this->benchmark->elapsed_time('jd_order_begin') . "秒");
        return $is_success;
    }
```
## 其他问题
1. 需要尽量提升mysql innodb引擎的性能
2. 代码是基于ci_2.0，ci_2.0的insert_battch有个小bug，返回的affect_rows最大是100..
3. 注意php的内存限制及执行环境
