
---
title: JAVA适配器模式-2016年上半半年考试真题
date: 2016-11-07 12:28:59
categories: 设计模式
tags:
 - 适配器模式
 - 软件设计师
 - JAVA
---
 ### 地址信息类 要求扩展 Dutch（荷兰）语言###
-----------------
 ### 现采用适配器模式（adapter）实现该要求： ###


 <!-- more -->

```java
class Address{
  public void street(){System.out.println("正常街道");}
  public void zip(){System.out.println("正常邮编");}
  public void city(){System.out.println("正常地方");}
}

class DutchAddress{

  public void straat(){System.out.println("荷兰语街道");}
  public void postcode(){System.out.println("荷兰语邮编");}
  public void plaats(){System.out.println("荷兰语地方");}
}

class DutchAddressAdapter extends DutchAddress{
  private Address address;
  public DutchAddressAdapter(Address addr){
    this.address=addr;
  }

  public void straat(){
    this.address.street();
  }
  public void postcode(){
    this.address.zip();
  }
  public void plaats(){
    this.address.city();
  }

}


public class Test {
  public static void main(String[] args){
    Address addr=new Address();
    DutchAddress addrAdapter=new DutchAddressAdapter(addr);
    System.out.println("\n THE DUCTH ADDRESS\n");
    testDutch(addrAdapter);
  }

  static void testDutch(DutchAddress addr){
    addr.straat();
    addr.postcode();
    addr.plaats();
  }

}
```
### 类图 ###

![适配器模式][1]


[1]: /img/patterndesign/adapter.png
