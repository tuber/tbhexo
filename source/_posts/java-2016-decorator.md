---
title: JAVA 装饰模式-2016软件设计师下半年考试真题
date: 2016-11-15 11:18:37
tags:
 - 装饰模式
 - 软件设计师
 - JAVA
---

## 题目要求：打印发票头、内容、底部的要求 ##

#### BTW,八成这次软考又挂了... ####

<!-- more -->

```java
    class Invoice{
      public void printInvoice(){
        System.out.println("this is content");
      }
    }

    class Decorator extends Invoice{
      protected Invoice ticket;
      public Decorator (Invoice t){
        ticket=t;
      }
      public void printInvoice(){
        if(ticket!=null){
          ticket.printInvoice();
        }
      }
    }

    class HeaderDecorator extends Decorator{

      public HeaderDecorator(Invoice t){
        super(t);
      }
      public void printInvoice(){
        System.out.println("this is the header");
        super.printInvoice();
      }

    }

    class FooterDecorator extends Decorator{

      public FooterDecorator(Invoice t){
        super(t);
      }
      public void printInvoice(){

        super.printInvoice();
        System.out.println("this is the footer");
      }

    }



    public class zhuangshi {

      public static void main(String[] args){
        Invoice t =new Invoice();
        Invoice ticket;
        ticket=new FooterDecorator(new HeaderDecorator(t));
        ticket.printInvoice();
        System.out.println("=====================");
        ticket=new FooterDecorator(new HeaderDecorator(new Decorator(null)));
        ticket.printInvoice();
      }

    }

```
## 这么写也行 ，不知道哪个算标准答案##
```java
     Invoice t =new Invoice();
      Invoice ticket;
      ticket=new HeaderDecorator(new FooterDecorator(t));
      ticket.printInvoice();
      System.out.println("=====================");
      ticket=new HeaderDecorator(new FooterDecorator(null));
      ticket.printInvoice();
```

## 结果 ##
![图片描述][1]


  [1]: /img/patterndesign/zhuangshi.png
