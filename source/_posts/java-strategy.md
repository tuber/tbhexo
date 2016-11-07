---
title: JAVA策略模式-2015年下半年考试真题
date: 2016-11-07 12:18:59
categories: 设计模式
tags:
 - 策略模式
 - 软件设计师
 - JAVA
---

 ### 大型商场要求商店有三种策略[原价、打折、满减]###
-----------------
 ### 现采用策略模式（strategy）实现该要求： ###

 <!-- more -->

```java
//枚举三种策略
enum TYPE {NORMAL,CASH_DISCOUNT,CASH_RETURN}

interface CashSuper{
  public double acceptCash(double money);
}

//原价
class CashNormal implements CashSuper{
  public double acceptCash(double money){
    return money;
  }
}

//打折
class CashDiscount implements CashSuper{
  private double discountRate;

  public CashDiscount(double discountRate){
    this.discountRate=discountRate;

  }
  public double acceptCash(double money){
    return money*discountRate;
  }
}

//满减
class CashReturn implements CashSuper{
  private double moneyCondition;
  private double moneyReturn;

  public CashReturn(double moneyCondition,double moneyReturn){

    this.moneyCondition=moneyCondition;
    this.moneyReturn=moneyReturn;

  }
  public double acceptCash(double money){

    double result =money;
    if(money>=moneyCondition){
      result=money-moneyReturn;
    }
    return result;
  }
}

// 策略
public class CashContent {
  private CashSuper sc;
  private TYPE t;

  public CashContent (TYPE t){
      switch (t){
      case NORMAL:
      sc=new CashNormal();
      break;

      case CASH_DISCOUNT:
      sc=new CashDiscount(0.9);
      break;


      case CASH_RETURN:
      sc=new CashReturn(300.00,50.00);
          break;
    }
  }

//实现
  public static void main(String[] argus){
    CashContent cc=new CashContent(TYPE.CASH_RETURN);
    //CashContent cc=new CashContent(TYPE.CASH_DISCOUNT);
    //CashContent cc=new CashContent(TYPE.NORMAL);

    System.out.println(cc.sc.acceptCash(900));//结果为900-50=850.0

  }
}


```
### 类图 ###

![策略模式][1]


[1]: /img/patterndesign/strategy.png
