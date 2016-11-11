---
title: java桥接模式-软件设计师考试2013下半年
date: 2016-11-11 09:22:03
categories: 设计模式
tags:
 - 桥接模式
 - 软件设计师
 - JAVA
---
### 类图及题目要求 ###

![桥接模式][1]


[1]: /img/patterndesign/qiaojie.png

<!-- more -->


### 代码可以运行 ###

```java



interface Drawing{
  public void drawLine(double x1,double y1,double x2,double y2);
  public void drawCircle(double x,double y,double r);
}

class V1Drawing implements Drawing{
  public void drawLine(double x1,double y1,double x2,double y2){DP1.draw_a_line(x1,y1,x2,y2);}
  public void drawCircle(double x,double y,double r){DP1.draw_a_circle(x, y, r);}
}
class V2Drawing implements Drawing{
  public void drawLine(double x1,double x2,double y1,double y2){DP2.drawLine(x1,y1,x2,y2);}
  public void drawCircle(double x,double y,double r){DP2.drawcircle(x, y, r);}
}

class DP1{
  static public void draw_a_line(double x1,double y1,double x2,double y2){
    System.out.println("DP1 画的线");
  }
  static public void draw_a_circle(double x,double y,double r){
    System.out.println("DP1 画的圆");
  }
}
class DP2{
  static public void drawLine(double x1,double y1,double x2,double y2){
    System.out.println("DP2 画的线");
  }
  static public void drawcircle(double x,double y,double r){
    System.out.println("DP2 画的圆");
  }
}



abstract class Shape{

  private Drawing _dp;

  public Shape(Drawing dp){
    this._dp=dp;
  }

  abstract public void draw();

  public void drawLine(double x1,double y1,double x2,double y2){

    this._dp.drawLine(x1,x2,y1,y2);
  }
  public void drawCircle(double x,double y,double r){this._dp.drawCircle(x,y,r);}

}
class Rectangle extends Shape{

  private double _x1,_x2,_y1,_y2;


  public Rectangle(Drawing dp,double x1,double x2,double y1,double y2){
    super(dp);
    this._x1=x1;
    this._x2=x2;
    this._y1=y1;
    this._y2=y2;

  }
  public void draw(){
    System.out.println("画长方形"+this._x1+this._x2+this._y1+this._y2);
    drawLine(_x1,_x2,_y1,_y2);
    }
}
class Circle extends Shape{
  private double _x,_y,_r;
  public Circle (Drawing dp,double x,double y,double r){
    super(dp);
  }
  public void draw(){drawCircle(_x,_y,_r);}
}

public class qiaojie {

  public static void main(String[] args){
    V1Drawing v1=new V1Drawing();
    V2Drawing v2=new V2Drawing();
    Rectangle r1=new Rectangle(v1,1.0,2.0,3.0,4.0);
    r1.draw();
    v2.drawCircle(2.1, 4.5, 5.6);
    v2.drawLine(2.4, 3.3 ,4.2, 5.1);

  }
}
```
