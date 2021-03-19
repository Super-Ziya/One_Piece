## Android shape属性学习笔记

- Corners：Corners标签用来字义圆角，其中radius与其它四个不能共同使用

```xml
<corners    //定义圆角   
    android:radius="dimension"      //全部的圆角半径   
    android:topLeftRadius="dimension"   //左上角的圆角半径   
    android:topRightRadius="dimension"  //右上角的圆角半径   
    android:bottomLeftRadius="dimension"    //左下角的圆角半径   
    android:bottomRightRadius="dimension" />    //右下角的圆角半径
```

- Solid：用以指定内部填充色，只有一个属性 `<solid  android:color="color" />` 
- grandient：定义渐变色，可以定义两色渐变和三色渐变，及渐变样式

```xml
<gradient  
    android:type=["linear" | "radial" | "sweep"] //共有3中渐变类型，线性渐变（默认）/放射渐变/扫描式渐变   
    android:angle="integer"     //渐变角度，必须为45的倍数，0为从左到右，90为从上到下，对线性渐变有效  
    android:centerX="float"     //渐变中心X的相当位置，范围为0～1，放射渐变时有效   
    android:centerY="float"     //渐变中心Y的相当位置，范围为0～1，放射渐变时有效 
    android:startColor="color"   //渐变开始点的颜色   
    android:centerColor="color"  //渐变中间点的颜色，在开始与结束点之间   
    android:endColor="color"    //渐变结束点的颜色   
    android:gradientRadius="float"  //渐变的半径，只有当渐变类型为radial时才能使用   
    android:useLevel=["true" | "false"] /> //使用LevelListDrawable时就要设置为true。设为false时才有渐变效果   
```

<img src="Image.assets\shape渐变.png" alt="shape渐变" style="zoom: 67%;" />

- stroke：描边属性，可以定义描边的宽度，颜色，虚实线

```xml
<stroke        
    android:width="dimension"   //描边的宽度   
    android:color="color"   //描边的颜色   
    // 以下两个属性设置虚线   
    android:dashWidth="dimension"   //虚线的宽度，值为0时是实线   
    android:dashGap="dimension" />      //虚线的间隔  
```

- Shape的属性（rectangle、oval、line、ring），矩形，椭圆形，线形和环形