---
title: php捕获fatal error
date: 2017-11-17 19:06:12
categories: PHP
tags:
 - 异常
---

用到laravel定时任务的时候，由于类似conf配置问题，无法捕捉到错误，采取的方式如下

{% codeblock lang:php %}
<?php
function fatal_handler() {
    $errfile = "unknown file";
    $errstr  = "shutdown";
    $errno   = E_CORE_ERROR;
    $errline = 0;

    $error = error_get_last();

    if($error){
      //发送邮件队列也可以
        file_put_contents('./testerror11.txt', json_encode($error));
    }
}
register_shutdown_function("fatal_handler");

try{
    $db=new db();
}catch(Exception $e){

echo $e->error_msg();
}

{% endcodeblock %}
