## Android群英传学习笔记

### 一、Android体系与系统架构

#### 1、Android系统架构

> Linux层：Android最低层最核心的部分。Linux层包含了Android系统的核心服务，包括硬件驱动、进程管理、安全系统，等等。
>
> Dalvik 与ART：Dalvik包含了一整套的Android运行环境虚拟机，每个App都会分配Dalvik虚拟机来保证互相之间不受干扰，并保持独立。它的特点是在运行时编译。而在Android 5.X版本开始，ART模式已经取代了Dalvik，ART采用的是安装时就进行编译，以后运行时就不用编译了。当然，对在其虚拟机环境中运行的大部分App来说，它们都运行着同样的代码。
>
> Framework
>
> Standard libraries：包含的是Android 中的一些标准库，就是开发者在开源环境中可以使用的开发库。
>
> Application：使用NDK开发和Java开发的App ，不管是哪种App，它们都有Android Manifest文件、Dalvik Classes、Resource Bundle这几个东西，这些就是我们解压Apk后的文件。

#### 2、应用运行时上下文对象

Activity,Service、Application都是继承自Context。

Android应用程序会在创建Application、Activity、Service时创建应用上下文Context。

创建Context的时机就是在创建Context的实现类的时候。当应用程序第一次启动时，Android系统都会创建一个Application对象，同时创建Application Context，所有的组件都共同拥有这样一个Context对象，这个应用上下文对象贯穿整个应用进程的生命周期，为应用全局提供了功能和环境支持。而创建Activity和Service组件时，系统也会给它们提供运行的上下文环境，即创建Activity 实例、Service实例的Context对象。所以在Activity中获取Context对象时，可以直接使用this，而在匿名内部类中,就必须指定XXActivity.this才可以获得该Activity的Context对象。当然也可以通过getApplicationContext() 方法来获取整个App的Context，但是通过getApplicationContext() 方法获得的是整个应用的上下文引用，这与某个组件的上下文引用，在某些时候还是有区别的。

#### 3、Android系统源代码目录与系统目录

**（1）Android系统源代码目录**

> Android作为手机操作系统，需要将源代码编译后才能使用，Eclipse、Android Studio这些属于开发IDE，也就是我们的集成开发环境，它体现的是一种简化计算机与开发者的交互，然而当你接触的程序架构越来越丰富，在了解越来越深入后，你就会发现，很多事情，IDE是无法完成的，比如自动化编译、定制编译、版本控制、自动测试等。因此Android与很多语言一样，引入了Makefile机制。那么Makefile到底有什么好处呢？我们先看一下对Makefile的解释：一个像Android这样的大型工程，它的源文件不计其数，不同的功能、模块，按类型分别放置在不同的目录中，这些模块通常会有一个叫Makefile的文件来进行管理。它定义了一系列的规则来指定模块，哪些文件需要编译，以及这些文件该按照怎样的顺序去编译。甚至，它还可以配置更复杂的功能操作，比如定义编译规则，打包规则等，因为Makefile就像一个shell脚本，不仅可以使用自己的语法，也能调用操作系统的命令。

**（2）Android系统目录**

> /system
>
> - /system/app/：这里面放的是一些系统的App。
> - /system/bin/：这里面主要放的是Linux自带的组件。
> - /system/build.prop：这里记录的是系统的属性信息。
> - /system/fonts/：系统字体存放目录root后可下载TTF格式字体替换原字体。
> - /system/framework/：系统的核心文件、框架层。
> - /system/lib/：存放几乎所有的共享库(.so) 文件。
> - /system/media/：该目录用来保存系统提示音、系统铃声。
> - /system/usr/：该目录用来保存用户的配置文件，如键盘布局、共享、时区文件等。
>
> /data
>
> - /data/app/：data目录包含了用户的大部分数据信息。其中，/data/app/这个 目录包含了用户安装的App或者升级的App。
> - /data/data/：这个目录包含了App的数据信息、文件信息、数据库信息等，以包名的方式来区分各个应用。
> - /data/system/：这个目录包含了手机的各项系统信息。
> - /data/misc/这个目录保存了大部分的Wi-Fi、VPN信息。

### 二、Android开发工具

#### 1、Android Studio 高级使用技巧

导入项目Gradle版本不一致：

首先在本地用当前版本的Gradle创建-一个 正常的项目，保证可以编译通过即可。然后,用地项目中的“gradle" 文件夹和“build.gradle" 文件，去替换要导入项目中的这两个文件夹，接下来，再打开这样的项目，就可以使用本地的Gradle对项目进行编译了。

#### 2、ADB命令使用技巧

**（1）ADB基础**

- ADB —— Android Debug Bridge。借助这样一个工具，可以用电脑来操纵手机
- ADB工具位于SDK的platform-tools目录下，因此在命令行中使用ABD的时候，需要通过cd命令，切换到该目录下，或者将platform-tools的路径添加到系统环境变量中，这样就可以直接使用了。配置好后，在命令行中输入以下命令 `adb version` ，显示版本内容即配置完成
- 如果是Windows系统，则还需要下载对应的手机驱动。Linux 系统则不需要下载。手机驱动可以靠各种手
  机助手来帮忙，这些手机助手也是使用ADB来实现它的功能的，下面我们再来设置一下手机端，进入开发者选项后，就可以选择“USB Debug”了。这时候，如果你的手机连接了电脑，那么手机界面上会显示“是否需要对这台电脑授权”的选项，选择“OK"后，电脑就通过ADB连接上了这台手机。
- 然后就可以使用shell命令。前面说了，Android其实就是基于Linux开发的，所以这里出现Shell也不奇怪。在进入Shell之后，我们就可以使用很多Linux下的Shell命令。

**（2）ADB常用命令**

- 显示系统中全部Android平台： `android list targets` 
- 安装Apk程序之Install： `adb insatll -r F:\Test.apk` 
- 安装Apk程序之Push： `adb push D:\Test.apk/system/app`  
  - 以上两种方法都可以安装上Apk，但是Adb Install是将Apk安装到data/data目录下，作为普通的用户应用程序。而Adb Push 不是安装命令，它是将一个文件写入手机存储系统。因此只要拥有相应的权限，就可以把任何Apk放到任何目录下，甚至是放到System目录下，成为一个系统应用程序。可以看出，Adb Push不仅可以安装Apk，它最大的用处还是向手机写入文件。
- 向手机写入文件： `adb push D:\file.txt/system/temp/` 
- 从手机获取文件： `adb push /system/temp/D:\file.txt` 

**（3）通过Logcat来查看Log，Grep（过滤）命令需要在Linux下使用**

- 查看Log： `adb shell` `logcat | grep "abc"` 
- 删除应用： `adb remount ` ( 重新挂载系统分区，使系统分区重新可写)。 `adb shell` `rm *.apk` 
- 查看系统盘： `adb shell df` 
- 输出所有已安装应用： `adb shell pm list packages -f` 
  - 该命令同样可以在Linux使用Grep来过滤结果
- 模拟按键输入： `adb shell input keyevent 3` 
  - 3是要执行的Keyevent的Code
- 模拟滑动输入： `adb shell input touchscreen 18 665 18 350 [<x1> <y1> <x2> <y2>]` 
- 查看运行状态： `adb shell dumpsys`  `dumpsys activity activities | grep "tencent"` 
  - 过滤”tencent“关键字
- Package管理信息：PM命令和Dumpsys命令一样，非常强大、复杂
  - 列出所有package： `pm list packages -f` 
- AM管理信息：这个命令很复杂，而且更强大
  - 启动一个Activity： `Adb shell am start -n 包名/包名+类名` 
- 录制屏幕： `adb shell screenrecord/sdcard/demo.mp4` 
- 重新启动： `adb reboot` 

**（4）ADB命令来源**

> CMD目录下的system\core\toolbox 和 TOOLBOX目录下的frameworks\base\cmds 就是ADB命令的来源

#### 3、模拟器的使用与配置

第三方模拟器：Genymotion

### 三、Android控件架构与自定义控件详解

#### 1、Android控件架构

在Android中，控件大致被分为两类，即ViewGroup控件与View控件。ViewGroup 控件作为父控件可以包含多个View控件，并管理其包含的View 控件。通过ViewGroup，整个界面上的控件形成了一个树形结构——控件树，上层控件负责下层子控件的测量与绘制，并传递交互事件。通常在Activity中使用的findViewByld() 方法，就是在控件树中以树的深度优先遍历来查找对应元素。在每棵控件树的顶部，都有一个ViewParent对象，这是整棵树的控制核心，所有的交互管理事件都由它来统一调度 和分配，从而可以对整个视图进行整体控制。

![view](Image.assets\view.jpg)

通常情况下，在Activity中使用setContentView() 方法来设置-一个布局，在调用该方法后，布局内容才真正地显示出来。

![UI界面架构](Image.assets\UI界面架构.jpg)

每个Activity都包含一个Window对象，在Android中Window对象通常由PhoneWindow来实现。PhoneWindow 将一个 DecorView 设置为整个应用窗口的根View。DecorView作为窗口界面的顶层视图，封装了一些窗口操作的通用方法。可以说，DecorView将要显示的具体内容呈现在了PhoneWindow上， 这里面的所有View 的监听事件，都通过WindowManagerService来进行接收，并通过Activity 对象来回调相应的onClickListener。在显示上，它将屏幕分成两部分，一个是TitleView，另一个是ContentView。ContentView是一个ID为content 的Framelayout ，activity_ main.xml 就是设置在这样-一个Framelayout 里。通过以上过程，我们可以建立起这样一个标准视图树。

![标准视图树](Image.assets\标准视图树.jpg)

视图树的第二层装载了一个LinearLayout，作为ViewGroup，这一层的布局结构会根据对应的参数设置不同的布局，如最常用的布局上面显示TitleBar下面是Content这样的布局。而如果用户通过设置requestWindowFeature (Window.FEATURE_NO_ TTLE) 来设置全屏显示，视图树中的布局就只有Content 了，这就解释了为什么调用requestWindowFeature() 方法一定要在调用setContentView() 方法之前才能生效的原因。由于每个Android版本对UI的修改都比较多，图这里只是比较粗略地显示了视图树的结构。

当程序在onCreate () 方法中调用setContentView() 方法后，ActivityManagerService会回调onResume() 方法，此时系统才会把整个DecorView 添加到PhoneWindow中，并让其显示出来，从而最终完成界面的绘制。

#### 2、View的测量

Android系统在绘制View前，也必须对View进行测量，即告诉系统该画一个多大的View。这个过程在onMeasure() 方法中进行。Android系统提供了一个设计短小精悍却功能强大的类一MeasureSpec 类，通过它来帮助我们测量View。MeasureSpec 是一个32位的int值，其中高2位为测量的模式，低30位为测量的大小，在计算中使用位运算的原因是为了提高并优化效率。

> 测量的模式可以为以下三种：

- EXACTLY：即精确值模式，当我们将控件的layout_width 属性或layout_height 属性指定为具体数值时，
  比如andorid:layout width="100dp"，或者指定为match_parent 属性时(占据父View的大小)，系统使用的是EXACTLY模式。
- AT MOST：即最大值模式，当控件的layout_width属性或layout_height 属性指定为wrap_content 时，控件大小一般随着控件的子控件或内容的变化而变化，此时控件的尺寸只要不超过父控件允许的最大尺寸即可。
- UNSPECIFIED：这个属性比较奇怪——它不指定其大小测量模式，View 想多大就多大，通常情况下在绘制自定义View时才会使用。

View类默认的onMeasure() 方法只支持 EXACTLY 模式，所以如果在自定义控件的时候不重写onMeasure() 方法的话，就只能使用EXACTLY模式。控件可以响应你指定的具体宽高值或者是match_parent 属性。而如果要让自定义View 支持wrap_content 属性，那么就必须重写onMeasure() 方法来指定wrap_content 时的大小。

通过MeasureSpec 这一个类，我们就获取了View 的测量模式和View想要绘制的大小。然后可以控制View最后显示的大小。

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```

系统最终会调用setMeasuredDimension(int measuredWidth, int measuredHeight) 方法将测量后的宽高值设置进去，完成测量工作。所以在重写onMeasure() 方法后，最终要做的工作就是把测量后的宽高值作为参数设置给setMeasuredDimension() 方法。

```java
@Override
protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec) {
    setMeasuredDimension(measureWidth(widthMeasureSpec),measureHeight(heightMeasureSpec));
}
```

在onMeasure() 方法中调用自定义的measureWidth() 方法和measureHeight() 方法分别对宽高进行重新定义，参数则是宽和高的MeasureSpec对象，MeasureSpec 对象中包含了测量的模式和测量值的大小。

> 自定义测量值。

- 从MeasureSpec对象中提取出具体的测量模式和大小

```java
int specMode = MeasureSpec.getMode(measureSpec);
int specSize = MeasureSpec.getSize(measureSpec);
```

- 接下来，通过判断测量的模式，给出不同的测量值。当specMode为EXACTLY时，直接使用指定的specSize即可，当specMode为其他两种模式时，需要给它一个默认的大小。特别地，如果指定wrap_content属性，即 AT_MOST模式，则需要取出我们指定的大小与specSize中最小的一个来作为最后的测量值，measureWidth()方法的代码如下所示

```java
private int measureWidth(int measureSpec){
    int result = 0;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(mcasureSpec);
    
    if (specMode == MeasureSpec.EXACTLY){
    	result = specSize;
    }else{
        result = 200;
        if (specMode == MeasureSpec.AT_MOST){
        	result = Math.min(result, specSize);
        }
    }
    return result;
}
```

- measureHeight() 方法与 measureWidth() 基本一致，通过这两个方法，就完成了对宽高值的自定义。当指定宽高属性为wrap_content属性时，如果不重写onMeasure() 方法，那么系统就不知道该使用默认多大的寸。因此，它就会默认填充整个父布局，所以重写onMeasure() 方法的目的，就是为了能够给View一个wrap_content属性下的默认大小。

#### 3、View的绘制

当测量好了一个View之后，就可以简单地重写onDraw() 方法，并在Canvas对象上来绘制所需要的图形。利用系统2D绘图API所必须要使用到Canvas对象。要想在Android的界面中绘制相应的图像，就必须在Canvas上进行绘制。Canvas就像是一个画板，使用Paint就可以在上面作画了。通常需要通过继承View并重写它的onDraw() 方法来完成绘图。

一般情况下，可以使用重写View类中的onDraw() 方法来绘图，onDraw() 中有一个参数，就是Canvas canvas对象。使用这个Canvas对象就可以进行绘图了，而在其他地方，通常需要使用代码创建一个Canvas对象：

 `Canvas canvas = new Canvas(bitmap);` 

当创建一个Canvas对象时，如果不传入一个bitmap对象，IDE编译虽然不会报错，但是一般不会这样做。这是因为传进去的bitmap与通过这个bitmap 创建的Canvas画布是紧紧联系在一起的，这个过程称之为装载画布。这个bitmap用来存储所有绘制在Canvas上的像素信息。所以当你通过这种方式创建了Canvas对象后，后面调用所有的Canvas.drawXXX方法都发生在这个bitmap上。如果在View类的onDraw()方法中，通过下面这段代码，可以了解到canvas与 bitmap直接的关系。首先在onDraw方法中绘制两个bitmap：

```java
canvas.drawBitmap(bitmap1,0,0,null);
canvas.drawBitmap(bitmap2,0,0,null);
```

对于bitmap2，将它装载到另一个Canvas对象中：

 `Canvas mCanvas = new Canvas(bitmap2);` 

在其他地方使用Canvas对象的绘图方法在装载bitmap2的Canvas对象上进行绘图： `mCanvas.drawXXX` 

通过mCanvas将绘制效果作用在了bitmap2上，再刷新View的时候，就会发现通过onDraw() 方法画出来的bitmap2已经发生了改变，这就是因为bitmap2承载了在mCanvas上所进行的绘图操作。虽然我们也使用了Canvas的绘制API，但其实并没有将图形直接绘制在onDraw() 方法指定的那块画布上，而是通过改变bitmap，然后让View重绘，从而显示改变之后的 bitmap。

#### 4、ViewGroup的测量

当ViewGroup的大小为wrap_content时，ViewGroup需要对子View进行遍历，以便获得所有子View的大小，从而来决定自己的大小。而在其他模式下则会通过具体的指定值来设置自身的大小。

ViewGroup在测量时通过遍历所有子View，从而调用子View的Measure方法来获得每一个子View的测量结果，前面所说的对View的测量，就是在这里进行的。

当子View测量完毕后，就需要将子View放到合适的位置，这个过程就是View的Layout过程。ViewGroup在执行Layout过程时，同样是使用遍历来调用子View的Layout方法，并指定其具体显示的位置，从而来决定其布局位置。

在自定义ViewGroup时，通常会去重写onLayout()方法来控制其子View显示位置的逻辑。同样，如果需要支持wrap_content属性，那么它还必须重写onMcasure()方法，这点与View是相同的。

#### 5、ViewGroup的绘制

ViewGroup通常情况下不需要绘制，因为它本身就没有需要绘制的东西，如果不是指定了ViewGroup的背景颜色，那么ViewGroup的onDraw() 方法都不会被调用。但是，ViewGroup会使用dispatchDraw() 方法来绘制其子View,其过程同样是通过遍历所有子View，并调用子View的绘制方法来完成绘制工作。

#### 6、自定义View

在自定义View时，通常会去重写onDraw() 方法来绘制View的显示内容。如果该View还需要使用wrap_content属性，那么还必须重写onMeasure() 方法。另外，通过自定义attrs属性，还可以设置新的属性配置值。

> 在View中通常有以下一些比较重要的回调方法：

- onFinishInflate()：从XML加载组件后回调。
- onSizeChanged()：组件大小改变时回调。
- onMeasure()：回调该方法来进行测量。 
- onLayout()：回调该方法来确定显示的位置。
- onTouchEvent()：监听到触摸事件时回调。

创建自定义View 的时候，并不需要重写所有的方法，只需要重写特定条件的回调方法即可。这也是 Android控件架构灵活性的体现。

> 通常情况下，有以下三种方法来实现自定义的控件。

- 对现有控件进行拓展
- 通过组合来实现新的控件
- 重写View来实现全新的控件

**（1）对现有控件进行拓展**

这是一个非常重要的自定义View方法，它可以在原生控件的基础上进行拓展，增加新的功能、修改显示的UI等。一般来说，可以在onDraw() 方法中对原生控件行为进行拓展。下面以一个TextView为例，来看看如何使用拓展原生控件的方法创建新的控件。比如想让一个TextView的背景更加丰富，给其多绘制几层背景。

原生的TextView使用onDraw() 方法绘制要显示的文字。当继承了系统的TextView之后，如果不重写其onDraw()方法，则不会修改TextView的任何效果。可以认为在自定义的TextView中调用TextView类的onDraw() 方法来绘制了显示的文字。

```java
@Override
protected void onDraw(Canvas canvas){
	super.onDraw(canvas);
}
```

程序调用super.onDraw(canvas) 方法来实现原生控件的功能，但是在调用super.onDraw() 方法之前和之后，都可以实现自己的逻辑，分别在系统绘制文字前后，完成自己的操作。

```java
@Override
protected void onDraw(Canvas canvas){
    //在回调父类方法前，实现自己的逻辑，对TextView来说即是在绘制文本内容前
    super.onDraw(canvas);
    //在回调父类方法后，实现自己的逻辑，对TextView来说即是在绘制文本内容后
}
```

以上就是通过改变控件的绘制行为创建自定义View 的思路。我们在构造方法中完成必要对象的初始化工作，如初始化画笔等：

```java
mPaintl = new Paint();
mPaint1.setColor(getResources().getColor(android.R.color.holo_blue_light));
mPaint1.setStyle(Paint.Style.FILL);
mPaint2 = new Paint();
mPaint2.setColor(Color.YELLOW);
mPaint2.setStyle(Paint.Style.FILL);
```

而代码中最重要的部分则是在onDraw() 方法中，为了改变原生的绘制行为，在系统调用super.onDraw(canvas) 方法前，也就是在绘制文字之下，绘制两个不同大小的矩形，形成一个重叠效果，再让系统调用super.onDraw(canvas)方法，执行绘制文字的工作。这样，我们就通过改变控件绘制行为，创建了一个新的控件：

```java
//绘制外层矩形
canvas.drawRect(
    0,
    0,
    getMeasuredWidth(),
    getMeasuredHeight(),
	mPaint1);
//绘制内层矩形
canvas.drawRect(
    10,
    10,
    getMeasuredWidth()-10,
    getMeasuredHeight() -10,
    mPaint2();
canvas.save();
//绘制文字前平移10像素
canvas.translate(10,0);
//父类完成的方法，即绘制文本
super.onDraw(canvas);
canvas.restore();
```

看一个稍微复杂一点的TextView。在前面一个实例中，我们直接使用了Canvas对象来进行图像的绘制，然后利用Android的绘图机制，可以绘制出更加复杂丰富的图像。比如可以利用LinearGradient Shader和Matrix来实现一个动态的文字闪动效果。

要想实现这一个效果，可以充分利用Android中Paint对象的Shader渲染器。通过设置一个不断变化的LinearGradient，并使用带有该属性的Paint对象来绘制要显示的文字。首先，在onSizeChanged() 方法中进行一些对象的初始化工作，并根据View的宽带设置一个LinearGradient渐变渲染器：

```java
@Override
protected void onSizeChanged(int w,int h,int oldw,int oldh){
    super.onSizeChanged(w,h,oldw,oldh);
    if(mViewWidth == 0){
        mViewWidth = getMeasuredWidth();
        if(mViewWidth > 0){
            mPaint = getPaint();
            mLinearGradient = new LinearGradient(
                0,
                0,
                mViewWidth,
                0,
                new int[]{
                    Color.BLUE,0xffffffff,
                    Color.BLUE},
                null,
                Shader.TileMode.CLAMP);
            mPaint.setShader(mLinearGradient);
            mGradientMatrix = new Matrix();
        }
    }
}

```

其中最关键的就是使用getPaint() 方法获取当前绘制TextView的Paint对象，并给这个Paint对象设置原生TextView没有的LinearGradient属性。最后，在onDraw() 方法中，通过矩阵的方式来不断平移渐变效果，从而在绘制文字时，产生动态的闪动效果：

```java
@Override
protected void onDraw(Canvas canvas){
    super.onDraw(canvas);
    if(mGradientMatrix != null){
        mTranslate += mViewWidth/5;
        if(mTranslate > 2 * mViewWidth){
        	mTranslate =- mViewWidth;
        }
        mGradientMatrix.setTranslate(mTranslate,O);
        mLinearGradient.setLocalMatrix(mGradientMatrix);
        postInvalidateDelayed(100);
    }
}
```

**（2）创建复合控件**

i、定义属性

创建复合控件可以很好地创建出具有重用功能的控件集合。这种方式通常需要继承一个合适的ViewGroup，再给它添加指定功能的控件，从而组合成新的复合控件。通过这种方式创建的控件，一般会给它指定一些可配置的属性，让它具有更强的拓展性。以TopBar为例。

很多应用程序都有一些共通的UI界面，通常情况下，这些界面都会被抽象出来，形成一个共通的UI组件。例如所有需要添加标题栏的界面都会引用这样一个TopBar，而不是每个界面都在布局文件中写这样一个TopBar。同时，
设计者还可以给TopBar增加相应的接口，不仅可以提高界面的复用率，更能在需要修改UI时，做到快速修改，而不需要对每个页面的标题栏都进行修改。

> 定义属性

- 在res资源目录的 values目录一个attrs.xml的属性定义文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="TopBar">
        <!--字符串-->
        <attr name="title" format="string"/>
        <!--尺寸值-->
        <attr name="titleTextSize" format="dimension"/>
        <!--颜色值-->
        <attr name="titleTextColor" format="color"/>
        <attr name="leftTextColor" format="color"/>
        <!--reference参考某一资源ID-->
        <attr name="leftBackground" format="reference|color"/>
        <attr name="leftText" format="string"/>
        <attr name="rightTextColor" format="color"/>
        <attr name="rightBackground" format="reference|color"/>
        <attr name="rightText" format="string"/>
    </declare-styleable>
</resources>
```

` <declare-styleable>` 标签：声明使用了自定义属性

​	 `name` 属性：确定引用名称

`<attr>` 标签：声明具体的自定义属性，例如标题文字字体、大小颜色、左右按钮文字颜色、背景字体等等

​	`format` 属性：指定属性类型，属性可以是颜色属性也可以是引用属性，使用|来分隔不同属性

在确定好属性后，创建一个自定义控件TopBar继承自ViewGroup，为了简单，这里继承RelativeLayout。在构造方法中，获取在XML布局文件中自定义的那些属性，即与我们使用系统提供的那些属性一样。

```java
TypedArray ta = context.obtainStyledAttributes(attrs,R.styleable.TopBar);
```

系统提供TypedArray这样的数据结构来获取自定义属性集，后面引用的 styleable的TopBar，就是在XML中通过`<declare-styleable name="TopBar">` 所指定的 `name` 名。接下来通过 `TypedArray` 对象的 `getString()` 、 `getColor()` 等方法获取这些定义的属性值。

```java
//通过这个方法，将在atts.xml中定义的 declare-styleable的所有属性的值存储到TypedArray中
TypedArray ta = context.obtainStyledAttributes(attrs,R.styleable.TopBar);
//从TypedArray中取出对应的值来为要设置的属性赋值
mLeftTextColor = ta.getColor(R.styleable.TopBar_leftTextColor,0);
mLeftBackground = ta.getDrawable(R.styleable.TopBar_leftBackground);
mLeftText = ta.getString(R.styleable.TopBar_leftText);

mRightTextColor = ta.getColor(R.styleable.TopBar_rightTextColor,0);
mRightBackground = ta.getDrawable(R.styleable.TopBar_rightBackground);
mRightText = ta.getString(R.styleable.TopBar_rightText);

mTitleTextSize = ta.getDimension(R.styleable.TopBar_titleTextSize,10);
mTitleTextColor= ta.getColor(R.styleable.TopBar_titleTextColor,O):
mlitle = ta.getString(R.styleable.TopBar_title);
//获取完TypedArray 的值后，一般要调用recyle方法来避免重新创建的时候的错误
ta.recycle();
```

ii、组合控件

UI模板的ToorBar实际上由三个控件组成，左边点击按钮MLeftButton，右边点击按钮mRightButton和中间的标题栏mTitleView。使用 `addView()` 方法动态添加三个控件到定义的TorBar模板中，设置前面获取到的具体属性值，例如标题文字颜色大小等。

```java
mLeftButton = new Button(context);
mRightButton = new Button(context);
mTitleView = new TextView(context);
//为创建的组件元素赋值，值就来源于引用的xml文件中给对应属性的赋值
mLeftButton.setTextColor(mLeftTextColor);
mLeftButton.setBackground(mLeftBackground);
mLeftButton.setText(mLeftText);

mRightButton.setTextColor(mRightTextColor);
mRightButton.setBackground(mRightBackground);
mRightButton.setText(mRightText);

mTitleView.setText(mTitle);
mTitleView.setTextColor(mTitleTextColor);
mTitleView.setTextSize(mTitleTextSize);
mTitleView.setGravity(Gravity.CENTER);
//为组件元素设置相应的布局元素
mLeftParamsnew LayoutParams(
	LayoutParams.WRAP_CONTENT,
	LayoutParams.MATCH_PARENT);
mLeftParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT,TRUE);
//添加到ViewGroup
addView(mLeftButton,mLeftParams);

mRightParams= new LayoutParams(
	LayoutParams.WRAP_CONTENT,
    LayoutParams.MATCH_PARENT);
mRightParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT,TRUE);
addView(mRightButton, mRightParams);

mTitlepParams=new LayoutParams(
	LayoutParams.WRAP_CONTENT,
	LayoutParams.MATCH PARENT);
mTitlepParams.addRule(RelativeLayout.CENTER_IN_PARENT,TRUE);
addView(mTitleView,mTitlepParams);
```

左右按钮点击事件不能直接在UI模板中添加具体的实现逻辑，只能通过接口回调将具体的实现逻辑交给调用者。

> 定义接口

在UI模板类中定义一个左右按钮点击的接口，并创建两个方法，分别用于左边按钮的点击和右边按钮的点击。

```java
//接口对象，实现回调机制，在回调方法中通过映射的接口对象调用接口中的方法而不用去考虑如何实现，具体的实现由调用者去创建
public interface topbarClickListener {
    //左按钮点击事件
    void leftClick();
    //右按钮点击事件
    void rightClick();
}
```

> 暴露接口给调用者

在模板方法中为左、右按钮增加点击事件，但不实现具体的逻辑，而是调用接口中相应的点击方法。

```java
//按钮的点击事件，不需要具体的实现，只需调用接口的方法，回调的时候，会有具体的实现
mRightButton.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
    	mListener.rightClick();
    }
}):
mLefButton.setOnClickListener(new OnClickL istener) {
    @override
    public void onClick(View v){
    	mListener.leftClick();
    }
});
//暴露一个方法给调用者来注册接口回调，通过接口来获得回调者对接口方法的实现
public void setOnTopbarClickListener(topbarClickListener mListener){
	this.mListener = mListener;
}
```

> 实现接口回调

调用者需要实现一个接口，并完成接口中的方法，并使用第二步中暴露的方法，将接口的对象传递进去，完成回调。通常情况下，可以使用匿名内部类的形式来实现接口中的方法。

```java
mTopbar.setOnTopbarClickListener(
    new TopBar.topbarClickListener(){
        @override
        public void rightClick(){
            Toast.makeText(TopBarTest.this,"right",Toast.LENGTH_SHORT).show();
        }
        @Override
        public void leftClick(){
	        Toast.makeText(TopBarTest.this,"left",Toast.LENGTH_SHORT).show();
        }
    });
```

除了通过接口回调的方式来实现动态的控制UI模板，同样可以使用公共方法来动态地修改UI模板中的UI，进一步提高了模板的可定制性。

```java
/**
 *设置按钮的显示与否通过id区分按钮，flag区分是否显示
 *
 *@param id		id
 *@param flag	是否显示
 */
public void setButtonVisable(int id,boolean flag){
    if (flag){
        if (id == 0){
        	mLeftButton.setVisibility(View.VISIBLE);
        }else{
        	mRightButton.setVisibility(View.VISIBLE);
        }
    }else{
        if(id == 0){
        	mLeftButton,setVisibility(View.GONE);
        }else {
        	mRightButton.setVisibility(View.GONE);
        }
    }
}
```

当调用者通过TopBar对象调用这个方法后，根据参数，调用者就可以动态地控制按钮的显示。

```java
//控制topbar上组件的状态
mTopbar.setButtonVisable(0, true);
mTopbar.setButtonVisable(1, false);
```

iii、引用UI模板

在引用前，需要指定引用第三方控件的名字空间。在布局文件中，有如下代码：

```xml
xmlns:android="http://lschemas.android.com/apk/res/android"
```

这行代码就是指定引用的名字空间 `xmlns` ，即 xml namespace。这里指定名字空间为“android”，在接下来使用系统属性时，才能使用 `“android:”` 来引用Android的系统属性。同样地，若使用自定义属性，需要创建自己的名字空间，在Android Studio中，第三方的控件都使用如下代码来引入名字空间。

```xml
xmlns:custom="http://schemas.android.com/apk/res-auto"
```

这里将引入的第三方控件的名字空间取名为custom，在XML文件中使用自定义属性时，可通过这个名字空间来引用。

```xml
<com.imooc.systemwidgct.TopBar
    android:id="@+id/topBar"
    android:layout_width="match parent"
    android:layout_height="40dp"
    custom:leftBackground="adrawablc/blue_button"
    custom:leftText="Back"
    custom:leftTextColor="#FFFFFF"
    custom:rightBackground="@drawable/blue_button"
    custom:rightlext="More"
    custom:rightlextColor="#FFFFFF"
    custom:title="自定义标题"
    custom:titleTextColor="#123412"
    custom:titleTextSize="10sp" />
```

使用自定义View与系统原生的View最大的区别：在申明控件时需要指定完整的包名，而在引用自定义的属性时，需要使用自定义的xmIns名字。

将该UI模板写到一个布局文件中。

```xml
<com.xys.mytopbar.Topbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:custom="http://schemas.android.com/apk/res-auto"
    android:id="@+id/topBar"
    android:layout_width="match_parent"
    android:layout_height"40dp"
    custom:leftBackground="@drawable/blue_button"
    custom:leftText="Back"
    custom:leftTextColor="#FFFFFF"
                                
    custom:rightBackground="@drawable/blue_button"
    custom:rightText="More"
    custom:rightTextColor="#FFFFFF"
                         
    custom:title="自定义标题"
    custom:titleTextColor="#123412"
    custom:titleTextSize="15sp">
    
</com.xys.mytopbar.Topbar>
```

通过<include>标签来引用该UI模板View。

```xml
<include layout="@layout/topbar"/>
```

**（3）重写View来实现全新的控件**

创建一个自定义View，难点在于绘制控件和实现交互，这也是评价一个自定义View优劣的标准之一。通常需要继承View类，并重写它的 `onDraw()` 、 `onMeasure()` 等方法来实现绘制逻辑，同时通过重写 `onTouchEvent()` 等触控事件来实现交互逻辑。还可以像实现组合控件方式，通过引入自定义属性，丰富自定义View的可定制性。

i、弧线展示图

![比例图](Image.assets\比例图.jpg)

该自定义View分为三个部分，分别是中间圆形、中间显示文字和外圈弧线。在 `onDraw()` 方法中一个个绘制即可。为了简单把View 绘制长度直接设置为屏幕的宽度。首先在初始化时设置好绘制三种图形的参数。

> 圆的代码：

```java
mCircleXY = length / 2;
mRadius = (float)(length * 0.5 / 2);
```

> 绘制弧线需要指定其椭圆的外接矩形：

```java
mArcRectF = new RectF(
	(float)(length * 0.1),
    (float)(length * 0.1),
    (float)(length * 0.9),
    (float)(length * 0.9));
```

> 绘制文字只需设置好文字起始位置即可

> onDraw() 方法绘制

```java
//绘制圆
canvas.drawCircle(mCirclexY,mCirclexY,mRadius,mCirclePaint);
//绘制弧线
canvas.drawArc(mArcRectF,270,mSweepAngle,false,mArcPaint);
//绘制文字
canvas.drawText(mShowText,0,mShowText.length(),mCirclexY,mCirclexY + (mShowTextSize / 4),mTextPaint);
```

不论多么复杂的图形、控件，都是由这些最基本的图形绘制出来的，关键在于如何去分解、设计这些图形，之后就只是对坐标的计算。

对于这个View，有一些方法可以让调用者来设置不同的状态值。

```java
public void setSwecpValue(float sweepValue){
    if(sweepValue != 0){
    	mSweepValue = sweepValue;
    }else {
    	mSweepValue = 25;
    }
    this.invalidate();
}
```

当用户不指定具体比例值时，可以默认设置为25，调用者可以通过如下代码设置相应比例值：

```java
CircleProgressView circle = (CircleProgressView)findViewByld(R.id.circle);
circle.setSweepValue(70);
```

ii、音频条形图

![音频图](Image.assets\音频图.jpg)

思路：绘制一个一个矩形，每个矩形之间偏移一点距离。

> 计算坐标的一种方法：

```java
@Override
protected void onDraw(Canvas canvas){
    super.onDraw(canvas);
    for (int i=0;i<mRectCount;i++){
        canvas.drawRect(
            (float)(mWidth * 0.4 / 2 + mRectWidth * i + offset),
            currentHeight,
            (float)(mWidth * 0.4 / 2 + mRectWidth * (i + 1)),
            mRectHeight,
            mPaint);
    }
}
```

通过循环创建小矩形，其中currentHeight是每个小矩形的高，通过横坐标的不断偏移，绘制出静态小矩形。再让这些小矩形的高度随机变化，通过Math.random() 方法随机改变高度值，并赋值给currentHeight。

```java
mRandom = Math.random();
float currentHeight = (float)(mRectHeight * mRandom);
```

这样就完成静态效果的绘制，只要在onDraw() 方法调用invalidate() 方法通知View进行重绘即可实现动态效果。不过不需要每次绘制完新矩形就通知View进行重绘，这样会因为刷新速度太快影响效果。可以进行View的延迟重绘。

```java
@Override
protected void onSizeChanged(int w,int h,int oldw,int oldh){
    super.onSizeChanged(w,h,oldw,oldh);
    mWidth = getWidth();
    mRectHeight = getHeight();
    mRectWidth = (int)(mWidth * 0.6 / mRectCount);
    mLinearGradient = new LinearGradient(
        0,
        0,
        mRectWidth,
        mRectHeight,
        Color.YELLOW,
        Color.BLUE,
        Shader.TileModc.CLAMP);
    mPaint.setShader(mLincarGradient);
}
```

在创建自定义View 时需要从一个基本效果开始，慢慢增加功能，绘制更复杂的效果。不论多么复杂的自定义View，一定是慢慢迭代起来的功能。

#### 7、事件拦截器 

>  触摸事件：捕触摸屏幕后产生的事件。

- 当点击一个按钮时，通常会产生两三个事件：按钮按下﹔不小心滑动一点；当手抬起时。

Android为触摸事件封装了一个类：MotionEvent，`onTouchEvent()` 方法的参数就是一个MotionEvent。触摸相关的方法的参数一般都含有MotionEvent。

> MotionEvent里面封装了：

- 触摸点坐标，通过 `event.getX()` 方法和 `event.getRawX()` 方法取出坐标点；
- 获得点击事件类型，通过不同的Action（如 `MotionEvent.ACTION_DOWN` 、 `MotionEvent.ACTION_MOVE` ）来进行区分，并实现不同的逻辑。
- ...

> 例如有布局结构：最外层 ViewGroup - A，中间层ViewGroup - B，底层View - M

- ViewGroup 重写方法： `dispatchTouchEvent(MotionEvent ev)` 、 `onInterceptTouchEvent(MotionEvent ev)` 、 `onTouchEvent(MotionEvent ev)` 
- View 重写方法： `onTouchEvent(MotionEvent ev)` 、 `dispatchTouchEvent(MotionEvent ev)` 

ViewGroup比View多一个方法 —— 

`onInterceptTouchEvent(MotionEvent ev)` ，是事件拦截核心方法。 `dispatchTouchEvent(MotionEvent ev)` 是事件分发的第一步，但一般不重写该方法

> 事件传递返回值：True - 处理，不继续；False - 不拦截，继续流程
>
> 事件处理返回值：True - 处理了，不用审核；False - 给上级处理

<img src="Image.assets\事件处理过程.png" alt="事件处理过程" style="zoom: 50%;" />

<img src="Image.assets\事件拦截.png" alt="事件拦截" style="zoom: 50%;" />

### 四、ListView使用技巧

#### 1、ListView常见优化技巧

**（1）设置项目间分割线**

-  `divider` 与 `dividerHeight` 属性

```xml
android:divider="@android:color/darker_gray"
android:dividerHeight="10dp"
```

- 当 `android:divider="@null"` 设置分割线为透明

**（2）取消ListView的Item点击效果**

- 点击ListView中的一项时，系统默认会出现一个点击效果，在Android5.x上是一个波纹效果，在Android5.X之下的版本是一个改变背景颜色的效果，可以通过修改 `listSelector` 属性来取消掉点击后的回馈效果。 	`android:listSelector="#0000000"` 也可使用Android 自带的透明色来实现。
   `android:listSelector="@android:color/transparent"` 

**（3）设置ListView显示在第几项**

- ListView以ltem为单位进行显示，默认显示在第一个Item，可指定具体显示的Item。
   `listViewsetSelection(N);` 其中N是显示的第N个Item。该方法类似 `scrollTo` ，是瞬间完成的移动。此外，还可使用如下代码实现平滑移动。

```java
mListView.smoothScrollBy(distance,duration);
mListView.smoothScrollByOffset(offset);
mListView.smoothScrollToPosition(index);
```

**（4）动态修改ListView**

- ListView中的数据在某些情况下需要变化，可以通过重新设置ListView的Adapter来更新ListView的显示，但也需要重新获取数据，相当于重新创建ListView，效率不高。可以使用更简单的方法来实现ListView的动态修改。

```java
mData.add("new");
mAdapter.notifyDataSetChanged();
```

- 当修改了传递给Adapter的映射List之后，调用Adapter的 `notifyDataSetChanged()` 方法，通知ListView更改数据源即可完成对ListView的动态修改。
- 使用 `mAdapter.notifyDataSetChanged()` 方法，必须保证传进Adapter的数据List是同一个List而不能是其他对象。

**（5）遍历LIstView所有item**

- ListView作为一个ViewGroup，常用 `getChildAt()` 来获取第i个子View。

```java
for (int i=0;i < mListView.getChildCount();i++) {
	View view = mListView.getChildAt(i);
}
```

**（6）处理空ListView**

- 当列表中无数据时，ListView 不会显示任何数据或提示，按照完善用户体验的需求，这里应该给以无数据的提示。ListView提供了一个方法 `setEmptyView()` , 该方法可以给ListView 设置一个在空数据下显示的默认布局。

```java
ListView listView = (ListView)findViewByld(R.id.listview);
listView setEmptyView(findViewById(R.id.empty_view));
```

**（7）滑动监听**

- 很多重写的ListView，基本都是在滑动事件的处理上下工夫。为了更加精确地监听滑动事件，通常还需要使用GestureDetector手势识别、VelocityTracker滑动速度检测等辅助类来完成更好的监听。

> OnTouchListener

OnTouchListencr是View中的监听事件，通过监听ACTION_DOWN、ACTION_MOVE、ACTION_UP 三个事件发生时的坐标，可以根据坐标判断滑动的方向，进行相应的逻辑处理。

```java
mListView.setOnTouchListener(new View.OnTouchListener(){
    @Override
    public boolean onTouch(View v,MotionEvent event){
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                //触摸时操作
                break;
            case MotionEvent.ACTION_MOVE:
                //移动时操作
                break;
            case MotionEvent.ACTION_UP:
                //离开时操作
                break;
        }
        return false;
    }
});
```

> OnScrollListener

OnScrollListener是AbsListView中的监听事件，封装了很多与ListView相关的信息，更加灵活。

```java
mListView.setOnScrollListencr(new OnScrollListener(){
    @Override
    public void onScrollStateChanged(AbsListView view. int scrollState){
        switch (scrollState){
            case onScrollListener.SCROLL_STATE_IDLE:
            	//滑动停止时
                Log.d("Test", "SCROLL_STATE_IDLE");
        		break;
            case OnScrollListencr.SCROLL_STATE_TOUCH_SCROLL:
                //正在滚动
                Log.d("Test", "SCROLL_STATE_TOUCH_SCROLL");
                break;
            case OnScrollListener.SCROLL_STATE_FLING:
                //手指抛动时即手指用力滑动
                //在离开后ListView由于惯性继续滑动
                Log.d("Test", "SCROLL_STATE_FLING");
                break;
        }
    }
    @Override
    public void onScroll(AbsListView view,
    					int firstVisibleItem,
        				int visibleItemCount,
        				int totalltemCount){
        //滚动时一直调用
        Log.d("Test", "onScroll");}
});
```

OnScrollListener中有两个回调方法 `onScrollStateChanged()` 和 `onScroll()` 

> `onScrollStateChanged()` ，根据参数 scrollState来决定其回调的次数

- scrollState有以下三种模式:
  -  `OnScrollListener.SCROLL_STATE_IDLE` 滚动停止时
  -  `OnScrollListener.SCROLL_STATE_TOUCH_SCROLL` 正在滚动时
  -  `OnScrollListener.SCROLL_STATE_FLING` 手指用力滑动，离开后ListView由于惯性继续滑动的状态。

没有做手指抛动的状态时，该方法只回调2次，否则回调3次。通常情况下会在该方法中通过不同的状态设置一些标志 Flag，来区分不同的滑动状态，供其他方法处理。

-  `onScroll()` ，在ListView滚动时会一直回调，后三个int类型参数，非常精确地显示了当前ListView 滚动的状态。
  - firstVisibleltem 当前能看见的第一个Item的ID（从0开始，当前能看见的 Item 数，包括没有显示完整的Item）
  - visibleltemCount 当前能看见的Item总数
  - totalltemCount 整个ListView 的Item总数

这些参数可以判断是否滚动到最后一行（当前可视的第一个ltem的ID加上当前可视Item的和等于Item总数时，即滚动到最后一行）

```java
if (firstVisibleltem + visibleltemCount == totallItemCount && totallItem - Count > O){
	//滚动到最后一行
}
```

或者判断滚动方向，通过成员变量lastVisibleItemPosition记录上次第一个可视的Item的ID并与当前可视Item的ID进行比较，即可知道当前滚动方向

```java
if(firstVisibleltem > lastVisibleltemPosition){
	//上滑
}else if (firstVisiblcltem < lastVisibleltemPosition){
	//下滑
}
lastVisibleltemPosition = firstVisibleItem;
```

ListView提供一些封装方法来获得当前可视的Item的位置等信息：

```java
//获取可视区域内最后一个Item的id
mListView.getLastVisiblePosition()
//获取可视区域内第一个Item的id
mListView.gctFirstVisiblePosition()
```

#### 2、ListView常见拓展

**（1）具有弹性的Listview**

Android 5.x 默认的ListView在滚动到顶端或底端时，只添加一个半月形阴影效果。而在 iOS系统中，列表滚动到底端或者顶端后会继续往下或者往上滑动一段距离。

通过重写ListView可以实现弹性效果，比如增加HeaderView或者使用ScrollView进行嵌套，不过可以使用一种非常简单的方法来实现，虽然不如其他方法可定制化高、效果丰富。

ListView源代码中有一个控制滑动到边缘的处理方法：

```java
protected boolean overScrollBy(int deltaX, int deltaY,
	int scrollX, int scrollY,
	int scrollRangeX, int scrollRangeY,
	int maxOverScrollX,int maxOverScrollY,
boolean isTouchEvent)
```

`maxOverScrollY` 注释：Number of pixels to overscroll by in either direction along the Yaxis。默认值是0，只要修改这个参数，就可实现弹性效果。重写该方法，将 maxOverScrollY 改为设置的值mMaxOverDistance。

```java
@Override
protected boolean overScrollBy(int deltaX, int deltaY,
                            int scrollX, int scrollY,
                            int scrollRangeX, int scrollRangeY,
                            int maxOverScrollX, int maxOverScrollY,
                            boolean isTouchEvent){
    return super.overScrollBy(deltax, deltaY,
                            scrollX, scrollY,
                            scrollRangeX, scrollRangeY,
                            maxOverScrollX, mMaxOverDistance,
                            isTouchEvent);
}
```

为满足多分辨率需求，在修改 maxOverScrollY 时，可通过屏幕的density来计算具体的值，让不同分辨率的弹性距离基本一致。

```java
private void initView(){
    DisplayMetrics metrics = mContext.getResources().getDisplayMetrics();
    float density = metrics.density;
    mMaxOverDistance = (int)(density * mMaxOverDistance);
}
```

**（2）自动显示、隐藏布局ListView**

让一个布局显示或者隐藏并带有动画效果，可通过属性动画实现，该效果关键在获得ListView的各种滑动事件。借助 View的 `OnTouchListener` 接口来监听ListView 的滑动，比较与上次坐标的大小，判断滑动方向，进而判断是否需要显示或隐藏对应布局。

首先给ListView增加一个HeaderView，避免第一个ltem被Toolbar遮挡。

```java
View header = new View(this);
header.setLayoutParams(new AbsListView.LayoutParams(
    AbsListView.LayoutParams.MATCH_PARENT,
    (int)getResources().getDimension(R.dimen.abc_action_bar_default_height_material)));
mListView.addHeaderView(header);
```

使用 abc_action_bar_default_height_material 属性获取系统Actionbar的高度，并设置给HeaderView。另外，定义一个 TouchSlop 变量用来获取系统认为的最低滑动距离（超过这个距离的移动，系统就将其定义为滑动状态）， `mTouchSlop = ViewConfiguration.get(this).getScaledTouchSlop();` 

判断滑动事件：

```java
View.OnTouchListener myTouchListener = new View.OnTouchListener(){
    @override
    public boolean onTouch(View v,MotionEvent event){
        switch(event.getAction){
            case MotionEvent.ACTION_DOWN:
            	mFirstY = event.getY();break;
            case MotionEvent.ACTION_MOVE:
            	mCurrentY = event.getY();
                if(mCurrentY - mFirstY > mTouchSlop){
                	direction = 0;//down
                }else if(mFirstY - mCurrentY > mTouchSlop){
                	direction = 1;// up
                }
                if(direction == 1){
                    if (mShow){
                        toolbarAnim(O);//hide
                        mShow = !mShow;
                    }
                }else if(direction == O){
                    if (!mShow){
                        toolbarAnim(1);//show
                        mShow = !mShow;
                    }
                }break;
            case MotionEvent.ACTION_UP:break;
        }
        return false;
    }
};
```

加上控制布局显示隐藏动画

```java
private void toolbarAnim(int flag)
    if(mAnimator != null && mAnimator.isRunning()){
    	mAnimator.cancel();
    }
    if(flag == 0){
	    mAnimator = ObjectAnimator.ofFloat(mToolbar,"translationY",mToolbar.getTranslationY(),0);
    }else{
    	mAnimator = ObjectAnimator.ofFloat(mToolbar,"translationY",mToolbar.getTranslationY(),-mToolbar.getHeight());
    }
    mAnimator.start();
}
```

Google 推荐 Toolbar 逐渐取代 ActionBar ，因为它更灵活。在使用时要使用的 theme 是 NoActionBar ，不然会引起冲突。

**（3）聊天ListView**

QQ、微信等聊天App的聊天界面，会展示至少两种布局（收到的消息和发送的消息），该效果是通过ListView实现的。

在定义 BaseAdapter 时，重写 `getView()` 方法，该方法用来获取布局，只需在获取布局时判断该获取哪种布局即可。ListView 在设计时考虑了这种情况，其提供两个方法。

```java
@Override
public int getItemViewType(int position){
	return type;
}

@Override
public int getViewTypeCount() {
	return number;
}
```

 `getltemViewType()`  方法用来返回第 position 个 Item 是何种类型，而 `getViewTypeCount()` 方法用来返回不同布局的总数。

首先实现两个布局。布局大同小异，只是方向上有区别。显示聊天信息内容的TextView使用9patch的图片

为了封装聊天内容，便于在Adapter中获取数据信息，封装了一个Bean保存聊天信息，声明set、get方法。

BaseAdapter使用 ViewHolder 模式提高 ListView 效率，在 `getView()` 方法中进行布局类型的判断，从而确定使用哪种布局。

```java
@Override
public int getltemViewType(int position){
    ChatltemListViewBean bean = mData.get(position);
    return bean.getType();
}

aOverride
public int getViewTypeCount(){
	return 2;
}

@Override
public View getView(int position,View convertView, ViewGroup parent){
    ViewHolder holder;
    if(convertView == null){
        if(getItemViewType(position) == 0){
            holder = new ViewHolder();
            convertView = mlnflater.inflate(R.layout.chat_item_itemin,null);
			holder.icon = convertView.findViewById(R.id.icon_in);
			holder.text = convertView.findViewById(R.id.text_in);
        }else{
			holder = new ViewHolder();
            convertView = mlnflater.inflate(R.layout.chat_item_itemout,null);
            holder.icon = convertVicw.findViewById(R.id.icon_out);
            holder.text = convertView.findViewById(R.id.text_out);
        }
		convertView.setTag(holder);
    }else {
	    holder = (ViewHolder)convertView.getTag();
    }
    holder.icon.setlmageBitmap(mData.get(position).getlcon());
    holder.text.setText(mData.get(position).getText());
    return convertView;
}
```

**（4）动态改变ListView**

> 动态改变点击 Item 的布局达到 Focus 的效果，一般有两种方法。

- 将两种布局写在一起，通过控制布局的显示、隐藏，达到切换布局的效果
- 在 `getView()` 的时候，通过判断加载不同的布局。

```java
public View getView(int position,View convertView,ViewGroup parent){
    LinearLayout layout = new LinearLayout(mContext);
    layout.setOrientation(LinearLayout.VERTICAL);
    
    if(mCurrentltem = position){
    	layout.addView(addFocusView(position));//点击布局
    }else{
    	layout.addView(addNormalView(position));//普通布局
    }
    return layout;
}
```

 `getView()` 在初始化时调用，点击 Item 时，没有再次调用 `getView()` 。要让 ListView 在点击后刷新一次。
 `notifyDataSetChanged()` 方法实现刷新布局。

```java
listView.setOnltemClickListener(new AdapterView.OnltemClickListener() {
    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id){
        adapter.setCurrentItem(position);
        adapter.notifyDataSetChanged();
    }
});
```

### 五、Android Scroll 分析

#### 1、滑动效果的产生

滑动一个View，本质上就是移动一个View。原理与动画效果的实现非常相似，都是通过不断地改变View的坐标来实现。实现View的滑动，必须监听用户触摸的事件，根据事件传入的坐标，动态且不断地改变View的坐标，实现View跟随用户触摸的滑动而滑动。

Android 的窗口坐标体系和屏幕的触控事件 —— MotionEvent

**（1）Android坐标系**

<img src="Image.assets\坐标系.png" alt="坐标系" style="zoom:50%;" />

系统提供 `getLocationOnScreen(int location[])` 方法获取 Android 坐标系中点的位置，即该视图左上角在Android 坐标系中的坐标。在触控事件中使用 `getRawX()` ， `getRawY()` 方法所获得的坐标也是 Android 坐标系中的坐标。

**（2）视图坐标系**

视图坐标系描述子视图在父视图中的位置关系。与 Android 坐标系类似，视图坐标系同样是以原点向右为X轴正方向，以原点向下为Y轴正方向

<img src="Image.assets\视图坐标系.png" alt="视图坐标系" style="zoom:50%;" />

在触控事件中，通过 `getX()` 、 `getY()` 所获得的坐标就是视图坐标系中的坐标。

**（3）触控事件 —— MotionEvent**

> MotionEvent 封装一些常用的事件常量

```java
//单点触摸按下动作
public static final int ACTION_DOWN = 0;
//单点触摸离开动作
public static final int ACTION_UP = 1;
//触摸点移动动作
public static final int ACTION_MOVE = 2;
//触摸动作取消
public static final int ACTION_CANCEL = 3;
//触摸动作超出边界
public static final int ACTION_OUTSIDE = 4;
//多点触摸按下动作
public static final int ACTION_POINTER_DOWN = 5;
//多点离开动作
public static final int ACTION_POINTER_UP = 6;
```

通常情况下会在 `onTouchEvent(MotionEvent event)` 方法中通过 `event.getAction()` 方法获取触控事件的类型，并使用 switch-case 方法进行筛选，代码模式基本固定。

```java
@Override
public boolean onTouchEvent(MotionEvent event){
    //获取当前输入点的X、Y坐标（视图坐标）
    int x = (int)event.getX();
    int y = (int)event.getY();
    switch (event.getAction){
        case MotionEvent.ACTION_DOWN:
            //处理输入的按下事件
            break;
        case MotionEvent.ACTION_MOVE:
            //处理输入的移动事件
            break;
        case MotionEvent.ACTION_UP:
            //处理输入的离开事件
            break;
    }
    return true;
}
```

不涉及多点操作的情况下，通常可以使用以上代码来完成触控事件的监听。

Android 系统提供非常多的方法来获取坐标值、相对距离等。

<img src="Image.assets\获取距离.png" alt="获取距离" style="zoom:50%;" />

> View 提供的获取坐标方法

-  `getTop()` 获取View自身顶边到其父布局顶边的距离
-  `getLeft()` 获取View自身左边到其父布局左边的距离
-  `getRight()` 获取View自身右边到其父布局左边的距离
-  `getBottom()` 获取View自身底边到其父布局顶边的距离

> MotionEvent提供的方法

-  `getX()` 获取点击事件距离控件左边的距离，即视图坐标
-  `getY()` 获取点击事件距离控件顶边的距离，即视图坐标
-  `getRawX()` 获取点击事件距离整个屏幕左边的距离，即绝对坐标
-  `getRawY()` 获取点击事件距离整个屏幕顶边的距离，即绝对坐标

#### 2、实现滑动七种方法

滑动效果实现的思想基本是当触摸View时，系统记下当前触摸点坐标，当手指移动时，系统记下移动后的触摸
点坐标，获取到相对于前一次坐标点的偏移量，通过偏移量来修改View 的坐标，不断重复，实现滑动过程。

**（1）layout方法**

在View进行绘制时，会调用 `onLayout()` 方法设置显示的位置。同样可以通过修改 View 的 left、top、right、bottom四个属性来控制 View 的坐标。在每次回调 onTouchEvent 时，都获取一下触摸点的坐标。

```java
int x = (int)event.getX();
int y = (int)event.getY();
```

接着在 ACTION_DOWN 事件中记录触摸点的坐标。

```java
case MotionEvent.ACTION_DOWN:
    //记录触摸点坐标
    lastX = x;
    lastY = y;break;
```

最后在 ACTION_MOVE 事件中计算偏移量，将偏移量作用到 Layout 方法中，在目前 Layout 的 left、top、right、bottom 基础上，增加偏移量。

```java
case MotionEvent.ACTION_MOVE:
    //计算偏移量
    int offsetX = x - lastX;
    int offsetY = y - lastY;
    //在当前 left、top、right、bottom 的基础上加上偏移量
    layout(getLeft() + offsetX,
		    getTop() + offsetY,
    		getRight() + offsetX,
    		getBottom() + offsetY);break;
```

上面使用的是 `getX()` 、 `getY()` 方法来获取坐标值，即通过视图坐标来获取偏移量。同样可以使用 `getRawX()` 、 `gctRawY()` 来获取坐标，使用绝对坐标来计算偏移量。

**（2）offsetLeftAndRight() 与 offsetTopAndBottom()** 

该方法相当于系统提供的一个对左右、上下移动的API的封装。计算出偏移量后，只需使用如下代码就可完成View的重新布局，效果与使用Layout方法一样。

```Java
//同时对left和 right进行偏移
offsetLeftAndRight(offsetX);
//同时对top和bottom进行偏移
offsetTopAndBottom(offsctY);
```


这里的 offsetX、offsetY 与在 Layout 方法中计算offset的方法一样。

**（3）LayoutParams**

LayoutParams 保存了一个View 的布局参数。可在程序中改变 LayoutParams 来动态修改布局的位置参数，达到改变View位置的效果。可以很方便在程序中使用 `getLayoutParams()` 来获取 View 的LayoutParams。计算偏移量的方法与在 Layout 方法中计算 offset 一样。获取偏移量后，通过 setLayoutParams 改变其 LayoutParams。

```java
LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
layoutParams.leftMargin = getLeft() + offsetX;
layoutParams.topMargin = getTop() + offsetY;
setLayoutParams(layoutParams);
```

通过 `getLayoutParams()` 获取 LayoutParams 时，需根据 View 所在父布局的类型设置不同的类型，比如将View 放在 LinearLayout 中，那么可以使用 LinearLayout.LayoutParams。前提是必须要有父布局，不然系统无法获取LayoutParams。

在通过改变LayoutParams来改变一个View的位置时，通常改变的是该View的 Margin 属性，除了使用布局的LayoutParams 之外，还可使用 ViewGroup.MarginLayoutParams 实现该功能。

```java
ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams)getLayoutParams();
layoutParams.leftMargin = getLeft() + offsetX;
layoutParams.topMargin = getTop() + offsetY;
setLayoutParams(layoutParams);
```


使用 ViewGroup.MarginLayoutParams 更加方便，不需要考虑父布局的类型。

**（4）scrollTo 与 scrollBy**

View 中的系统提供了 scrollTo、scrollBy 两种方式来改变一个View的位置。 `scrollTo(x, y)` 表示移动到一个具体的坐标点 (x, y)， `scrollBy(dx, dy)` 表示移动的增量为 dx、dy。scrollTo,、scrollBy 方法移动的是 View 的content，即让 View 的内容移动。在 ViewGroup 中使用 scrollTo、scrollBy 方法，移动的是所有子 View，在View中使用，移动的是 View 的内容。

手机屏幕如中空的盖板，盖板下面是一个巨大的画布，也就是显示的视图。调用 scrollBy 方法，可认为盖板在移动。

<img src="Image.assets\scrollBy.png" alt="scrollBy" style="zoom:75%;" />

 `scrollBy(20,10);` 

<img src="Image.assets\scrollBy2.png" alt="scrollBy2" style="zoom:75%;" />

要实现跟随手指移动而滑动的效果，必须将偏移量改为负值。

```java
int offsetX = x - lastX;
int offsetY = y - lastY;
((View)getParent()).scrollBy(-offsetX,-offsetY);
```

**（5）Scroller**

Scroller类与 scrollTo、scrollBy 方法十分相似。

假如完成这样一个效果：点击按钮，让一个 ViewGroup 的子View 向右移动100个像素。不管使用 scrollTo 还是 scrollBy 方法，子 View 的平移都是瞬间发生的，效果非常突兀。Google 建议使用自然的过度动画来实现。通过 Scroller 类可以实现平滑移动的效果。

Scroller类的实现原理与使用 scrollTo 和 scrollBy 方法实现子 View 跟随手指移动的原理基本类似。在 ACTION_MOVE 事件中不断获取手指移动的微小偏移量，将一段距离划分成了 N 个非常小的偏移量。在每个偏移量里面通过 scrollBy 方法进行瞬间移动，但在整体上可以获得一个平滑移动的效果。利用了人眼的视觉暂留特性。

> 实例：让子View跟随手指的滑动而滑动，在手指离开屏幕时，让子View平滑移动到初始位置（屏幕左上角）

- 使用Scroller

  - 初始化Scroller

    ```java
    //初始化Scroller
    mScroller = new Scroller(context);
    ```

  - 重写 `computeScroll()` 方法，实现模拟滑动

     `computcScroll()` 方法是使用 Scroller 类的核心，系统在绘制 View 时会在 `draw()` 方法中调用该方法。该方法实际上使用 scrollTo 方法。结合 Scroller 对象获取到当前的滚动值。可以通过不断瞬间移动一段小距离来实现整体上的平滑移动效果。computeScroll 可以利用如下模板代码实现。

    ```java
    @Override
    public void computeScroll(){
        super.computcScroll();
        //判断Scroller是否执行完毕
        if(mScroller.computeScrollOffset()){
        	((View)getParent()).scrollTo(
                mScroller.getCurrX(),
                mScroller.getCurrY());
            //通过重绘来不断调用computeScroll
            invalidate();
        }
    }
    ```

    Scroller 类提供了 `computeScrollOffset()` 方法判断是否完成整个滑动，提供 `getCurrX()` 、 `getCurrY()` 方法获得当前的滑动坐标。因为只能在 `computcScroll()` 方法中获取模拟过程中的 scrollX 和 scrollY 坐标，但 `computeScroll()` 方法不会自动调用，只能通过 `invalidate()` → `draw()` → `computeScroll()` 间接调用 `computeScroll()` 方法。当模拟过程结束后， `scroller.computeScrollOffset()` 方法会返回 false，从而中断循环，完成整个平滑移动过程。

  - startScroll 开启模拟过程

    在需要使用平滑移动的事件中使用 Scroller 类的 `startScroll()` 方法开启平滑移动过程。 `startScroll()` 方法有两个重载方法：

    -  `public void startScroll(int startX,int startY,int dx,int dy,int duration)` 
    -  `public void startScroll(int startX,int startY,int dx,int dy)` 

    它们的区别就是一个具有指定的持续时长，另一个没有。与在动画中设置 duration 和使用默认的显示时长是一个道理。其他四个坐标与其命名含义相同，是起始坐标与偏移量。获取坐标时使用 `getScrollX()` 和 `getScrollY()` 方法来获取父视图中 content 滑动到的点的坐标，要注意该值的正负。

    在构造方法中初始化 Scroller 对象，重写 View 的 `computeScroll()` 方法。在 onTouchEvent 中加一个ACTION_UP 监听手指离开屏幕的事件，在该事件中调用 `startScroll()` 方法完成平滑移动。

    ```java
    case MotionEvent.ACTION_UP:
        //手指离开时，执行滑动过程
        View viewGroup = ((View)getParent());
        mScroller.startScroll(
            viewGroup.getScrollX(),
            vicwGroup.getScrollY(),
            -viewGroup.getScrollX(),
            -viewGroup.getScrollY());
    	invalidate();
    	break;
    ```

    在 `startScroll()` 方法中，获取子 View 移动的距离 —— `getScrollX()` 、 `getScrollY()` ，并将
    偏移量设为其相反数，从而将子View滑动到原位置。使用 `invalidate()` 方法通知 View 重绘，调用 `computeScroll()` 的模拟过程。也可以给 `startScroll()` 方法增加一个 duration 参数设置滑动的持续时长。

**（6）属性动画**

第七章讲解

**（7）ViewDragHelper**

Google 在 support 库提供 DrawerLayout 和 SlidingPaneLayout 两个布局实现侧边栏滑动的效果。通过ViewDragHelper，基本可以实现各种不同的滑动、拖放需求。ViewDragHelper 功能强大，使用方法也最复杂。

> 实例：实现类似QQ滑动侧边栏的布局，初始显示内容界面，当用户手指滑动超过一段距离时，内容界面侧
> 滑显示菜单界面。

- 初始化ViewDragHelper

  ViewDragHelper 通常定义在一个 ViewGroup 内部，通过其静态工厂方法进行初始化。

   `mViewDragHelper = ViewDragHelper.create(this, callback);` 
  第一个参数是要监听的 View，通常是一个ViewGroup，即 parentView;
  第二个参数是一个 Callback 回调，该回调是整个 ViewDragHelper 的逻辑核心。

- 拦截事件

  重写事件拦截方法，将事件传递给 ViewDragHelper 处理。

  ```java
  @override
  public boolean onInterceptTouchEvent(MotionEvent ev){
  	return mViewDragHelper.shouldInterceptTouchEvent(ev);
  }
  
  @Override
  public boolean onTouchEvent(MotionEvent event){
      //将触摸事件传递给ViewDragHelper,此操作必不可少
      mViewDragHelper.processTouchEvent(event);
      return true;
  }
  ```

- 处理 `computeScroll()` 

  ViewDragHelper 内部也是通过 Scroller 实现平滑移动。模板代码：

  ```java
  @Override
  public void computeScroll(){
  	if(mViewDragHelper.continueSettling(true)){
  		ViewCompat.postInvalidateOnAnimation(this);
      }
  }
  ```

- 处理回调Callback

  创建一个 ViewDragHelper.Callback

  ```java
  private ViewDragHelper.Callback callback = new ViewDragHelper.Callback(){
      @override
      public boolcan tryCaptureView(View child,int pointerld){
      	return false;
      }
  };
  ```

  IDE自动重写 `tryCaptureView()` 方法。该方法可以指定在创建 ViewDragHelper 时，参数 parentView 中的哪一个子 View 可以被移动，例如自定义一个ViewGroup，里面定义了两个子 View —— MenuView 和MainView，指定如下代码时，只有 MainView 可以被拖动。

  ```java
  //何时开始检测触摸事件
  @Override
  public boolean tryCaptureView(View child,int pointerld){
      //如果当前触摸的child是mMainView时开始检测
      return mMainView == child;
  }
  ```

  具体滑动方法 —— `clampViewPositionVertical()` 和 `clampViewPositionHorizontal()` ，分别对应垂直和水平方向上的滑动。要实现滑动效果这两个方法必须重写。因为它默认返回值为0，即不发生滑动。

  ```java
  @Override
  public int clampViewPositionVertical(View child, int top, int dy){
  	return top;
  }
  
  @Override
  public int clampViewPositionHorizontal(View child, int left, int dx){
  	return left;
  }
  ```

   `clampViewPositionVertical(View child,int top,int dy)` 方法：
  	参数 top 代表在垂直方向上 child 移动的距离；
  	参数 dy 表示比较前一次的增量。

  同理 clampViewPositionHorizontal 也是类似。通常只需返回 top 和 left 即可，要更精确计算 padding 等属性时，要对 left 进行处理，返回合适的值。

  重写这三个方法，可以实现一个最基本的滑动效果。用手拖动 MainView 时，可以跟随手指的滑动而滑动。

  ```java
  private ViewDragHelper.Callback callback = new ViewDragHelper.Callback(){
      //何时开始检测触摸事件
      @Override
      public boolean tryCaptureView (View child, int pointerld){
          //如果当前触摸的 child 是 mMainView 时开始检测
          return mMainView == child;
      }
      
      @Override
      public int clampViewPositionVertical (View child,int top,int dy){
      	return 0;
      }
      
      @Override
      public int clampViewPositionHlorizontal (View child,int left,int dx){
      	return left;
      }
  };
  ```

  在手指离开屏幕后，子View滑动回初始位置。在 ViewDragHelper.Callback 中，系统提供 `onViewReleased()` 方法，该方法内部通过 Scroller 类实现的，这也是前面重写 `computeScroll()` 方法的原因。

  ```java
  //拖动结束后调用
  @Override
  public void onViewReleased(View relcasedChild,float xvel,float yvel){
      super.onViewReleased(releasedChild, xvel, yvel);
      //手指抬起后缓慢移动到指定位置
      if(mMainView.getLeft() < 500){
      	//关闭菜单
      	//相当于Scroller的startScroll方法
      	mViewDragHelper.smoothSlideViewTo(mMainView,0,0);
      	ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
      }else {
      	//打开菜单
      	mViewDragHelper.smoothSlideViewTo(mMainView,300,0);
      	ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
      }
  }
  ```

  设置让 MainView 移动后左边距小于500像素时使用 `smoothSlideViewTo()` 方法将 MainView 还原到初始状态，即(0,0)点。当左边距大于500时，将 MainView 移动到(300,0)，即显示MenuView。

自定义一个 ViewGroup 来完成实例。在自定义 ViewGroup 的 `onFinishInflate()` 方法中按顺序将子 View 分别定义成 MenuView和 MainView，并在 `onSizeChanged()` 方法中获得 View 的宽度。

```java
//加载完布局文件后调用
@Override
protected void onFinishInflateo(){
	super.onFinishInflate();
	mMenuView = getChildAt(0);
	mMainView = getChildAt(1);
}

@Override
protected void onSizeChanged(int w,int h,int oldw,int oldh){
	super.onSizeChanged(w,h,oldw,oldh);
	mWidth = mMenuView.getMeasuredWidth();
}
```

最后通过 ViewDragHelper 实现QQ侧滑功能。

> 在 ViewDragHelper.Callback 中，系统定义了大量的监听事件来处理各种事件：

-  `onViewCaptured()` 
  在用户触摸到View后回调。
-  `onViewDragStateChanged()` 
  在拖拽状态改变时回调，比如 idle，dragging 等状态。
-  `onViewPositionChanged()` 
  在位置改变时回调，常用于滑动时更改 scale 进行缩放等效果。

### 六、Android绘图机制与处理技巧

#### 1、屏幕的尺寸信息

**（1）屏幕参数**

> 屏幕大小

- 指屏幕对角线的长度，通常使用“寸”来度量

> 分辨率

- 分辨率是指手机屏幕的像素点个数，例如 720×1280 指宽有720个像素点，而高有1280个像素点

> PPI

- 每英寸像素（Pixels Per Inch) 又被称为DPI (Dots Per Inch)。由对角线像素点数除以屏幕大小得到，通常400 PPI 是非常高的屏幕密度

**（2）系统屏幕密度**

每个厂商的 Android 手机具有不同大小尺寸和像素密度的屏幕。系统定义了几个标准的 DPI 值，作为手机的固定DPI 。

<img src="Image.assets\DPI.png" alt="DPI"  />

**（3）独立像素密度**

​		由于各种屏幕密度不同，导致同样像素大小长度，在不同密度的屏幕上显示长度不同。因为相同长度的屏幕，高密度的屏幕包含更多的像素点。Android 系统使用 mdpi （密度值）为160的屏幕作为标准，在这个屏幕上 1px = 1dp。其他屏幕可以通过比例进行换算，如100dp的长度，在 mdpi 中为100px，在 hdpi 中为150px。各个分辨率直接的换算比例：ldpi : mdpi : hdpi : xhdpi : xxhdpi = 3 : 4 : 6 : 8 : 12

**（4）单位转换**

```java
/**
  *dp、sp转换为px的工具类
  */
public class DisplayUtil{
    /**
      *将px值转换为dip或dp值，保证尺寸大小不变
      *@param px Value
      *@param scale
      *		（DisplayMetrics类中属性 density）
      *@return
      */
    public static int px2dip(Context context,float pxValue){
        //获取手机屏幕参数，density是屏幕密度
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int)(pxValue / scale + 0.5f);
    }
    
    /**
    *将dip或dp值转换为px值，保证尺寸大小不变
    *@param dipValuc
    *@param scale
    *
    *		（DisplayMetrics类中属性density）
    *@return
    */
    public static int dip2px(Context context,float dipValue){
        //获取手机屏幕参数，density是屏幕密度
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int)(dipValue * scale + 0.5f);
    }
    
    /**
      *将px值转换为sp值，保证文字大小不变
      *@param pxValue
      *@param fontScale
      *		（DisplayMetrics类中属性scaledDensity）
      *@return
      */
    public static int px2sp(Context context,float pxValue){
        //获取手机屏幕参数，scaledDensity是缩放密度
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int)(pxValue / fontScale + 0.5f);
    }
    
    /**
      *将sp值转换为px值，保证文字大小不变
      *@param spValue
      *@param fontScale
      *		（DisplayMetrics 类中属性scaledDensity）
      *@return
      */
    public static int sp2px(Context context,float spValue){
        //获取手机屏幕参数，scaledDensity是缩放密度
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int)(spValue * fontScale + 0.5f);
    }
}
```

其中 density 是前面说的换算比例。这里使用公式换算方法进行转换。

系统提供TypedValue类帮助转换：

```java
/**
  *dp2px
  */
protected int dp2px(int dp){
    return (int)TypedValue.applyDimension(
            TypedValue.COMPLEX_UNIT_DIP,
            dp,
            getResources().getDisplayMetrics());
}

/**
  *sp2px
  */
protected int sp2px(int sp){
    return (int)TypedValue.applyDimension(
            TypedValue.COMPLEX_UNIT_SP,
            sp,
            getResources().getDisplayMetrics());
}
```

#### 2、2D绘图基础

系统通过 Canvas 对象提供了各种绘制图像的 API，如 drawPoint(点)、drawLine(线)、drawRect(矩形)、drawVertices(多边形)、drawArc(弧)、drawCircle(圆) 等。

> Paint 一些属性和对应功能：

-  `setAntiAlias();`  //设置画笔的锯齿效果
-  `setColor();`  //设置画笔的颜色
-  `setARGB();`  //设置画笔的A、R、G、B值
-  `setAlpha();`  //设置画笔的Alpha值
-  `setTextSize();`  //设置字体的尺寸
-  `setStyle();`  //设置画笔的风格（空心或实心)
-  `setStrokeWidth();`  //设置空心边框的宽度

> 矩形设置 Paint 的 Style 可画出空心 / 实心的矩形

- 设置空心： `paint.setStyle(Paint.Style.STROKE);` 

- 设置实心： `paint.setStyle(Paint.Style.FILL);` 

> 绘制点线

- 绘制点： `canvas.drawPoint(x,y,paint);` 

- 绘制直线： `canvas.draLine(startX,startY,endX,endY,paint);` 

- 绘制多条直线（折线）：

  ```java
  float[] pts={
  	startX1,startX1,endX1,endY1,
      ...
  	startXn,startYn,endYn,endYn};
  canvas.drawLines(pts, paint);
  ```

> 绘制图形

- 绘制矩形： `canvas.drawRect(left,top,right,bottom,paint);` 

- 绘制圆角矩形： `canvas.drawRoundRect(left,top,bottom,radiusX,radiusY,paint);` 

- 绘制圆： `canvas.drawCircle(circleX,circleY,radius,paint);` 

- 绘制弧形、扇形：

  ```java
  paint.setStyle(Paint.Style.STROKE);
  canvas.drawArc(left,top,right,bottom,
  				startAngle,sweepAngle,useCenter,paint);
  ```

  - 绘制孤形与扇形的区分是倒数第二个参数 useCenter 的区别，使用不同的 Paint.Style 和 useCenter 属性产生的不同效果，空心和实心和上面一样

  ```java
  Paint.Style.STROKE + useCenter(true);
  Paint.Style.STROKE + useCenter(false);
  ```

- 绘制椭圆： `canvas.drawOval(left,top,right,bottom,paint);` 

> 绘制文本

- 绘制文本： `canvas.drawText(text,startX,startY,paint);` 

- 在指定位置绘制文本：

  ```java
  canvas.drawPosText(text,
                      new float[]{X1,Y1,
                                  X2,Y2,
                                  Xn,Yn},
                      paint);
  ```

- 绘制路径：

  ```java
  Path path = new Path();
  path.moveTo(50,50);
  path.lineTo(100,100);
  path.lineTo(100,300);
  path.lineTo(300,50);
  canvas.drawPath(path, paint);
  ```

#### 3、Android XML绘图

XML 在 Android 系统中不仅是 Java 中的布局文件、配置列表。它可以是一张画、一幅图

**（1）Bitmap**

```xml
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
	android:src="@drawable/ic_launcher"/>
```

这样引用图片，可以将图片直接转成 Bitmap 在程序中使用

**（2）Shape**

shape 可以在 xml 中绘制各种形状

```xml
<shape
	xmins:android-http://schemas.android.com/apk/res/android
	//默认为rectangle
	android:shape=["rectangle"|"oval"|"line"|"ring"]>
    
    <corners //当shape="rectangle"时使用半径，会被后面的单个半径属性覆盖，默认为1dp
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer"/>
                                           
    <gradient //渐变
        android:angle="integer"
        android:centerX="integer"
        android:centerY="integer"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear"|"radial"|"sweep"]
        android:useLevel=["true"|"false"]/>
    
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer"/>
    
    <size //指定大小，一般用在imageview配合scaleType属性使用
        android:width="integer"
        android:height="integer"/>
    
    <solid //填充颜色
    	android:color="color"/>
    
    <stroke //指定边框
        android:width="integer"
        android:color="color"
        //虚线宽度
        android:dashWidth="integer"
        //虚线间隔宽度
        android:dashGap="integer"/>
</shape>
```

**（3）Layer**

Layer是在Photoshop中非常常用的功能，在Android中同样可以通过Layer实现类似图层的概念。

```xml
<?xml version="1.0" encoding "utf-8"?>
<layer-1ist xmlns:android="http:/lschemas.android.com/apk/res/android">
	<!--图片1-->
	<item
		android:drawable="@drawable/ic_launcher">
	<!--图片2-->
	<item
        android:drawablc="@drawable/ic_launcher"
        android:left="10.0dip"
        android:top="10.0dip"
        android:right="10.0dip"
        android:bottom="10.0dip"/>
</layer-list>
```

![图层](Image.assets\图层.png)

**（4）Selector**

Selector 帮助实现静态绘图中的事件反馈，通过给不同的事件设置不同的图像，从而在程序中根据用户输入，返回不同的效果。

```xml
<?xml version="1.0" encoding "utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!--默认时的背景图片-->
    <item android:drawable="@drawable/X1"/>
    <!--没有焦点时的背景图片-->
    <item android:state_window_focused="false"
        android:drawable="@drawable/x2"/>
    <!--非触摸模式下抉得焦点并单击时的背景图片-->
    <item android:state_focused="true"
        android:state_ressed="true"
        android:drawable="@drawable/X3"/>
    <!--触摸模式下单击时的背景图片-->
    <item android:state_focused"false"
        android:state_pressed="true"
        android:drawable="@drawable/X4"/>
    <!--选中时的图片背景-->
    <item android:state_selected="true"
        android:drawable="@drawable/X5"/>
    <!--获得焦点时的图片背景-->
    <item android:state_focused="true"
        android:drawable="@drawable/X5">
</selector>
```

通过配置不同的触发事件，Selector 可以自动选择不同的图片。特别是在自定义Button的时候，就不用再使用原生 Button 单调的背景，而使用 Selector 定制的背景。不用在程序中修改点击的逻辑，就能完美实现触摸反馈。通常情况下，上面这些方法都是可以共同作用的。例如 Selctor 中使用 Shape 作为它的 Item。

#### 4、Android 绘图技巧

**（1）Canvas**

>  `Canvas.save()`  ：字面上理解为保存画布。作用是将之前所有已绘制图像保存起来，让后续操作好像在一个新图层上操作一样
>
>  `Canvas.restore()`  ：可以理解为Photoshop中的合并图层操作。将在save() 之后绘制的所有图像与 save() 之前的图像进行合并
>
>  `Canvas.translate()`  ：画布平移，默认绘图坐标零点位于屏幕左上角,在调用 translate(x, y) 方法后，将原点 (0,0) 移动到 (x,y)。之后所有绘图操作以 (x,y) 为原点执行
>
>  `Canvas.rotate()`  ：画布翻转，将坐标系旋转一定角度

**（2）Layer图层**

在Android中使用 `saveLayer()` 方法创建一个图层，图层基于栈结构进行管理

Android 通过调用 `saveLayer()` 方法、 `savaLayerAlpha()` 方法将一个图层入栈，使用 `restore()` 方法、 `restoreToCount()` 方法将一个图层出栈。入栈时，后面所有操作都发生在该图层上，出栈时，会把图像绘制到上层 Canvas 上。

#### 5、Android图像处理之色彩特效处理

Android 对图片的处理，最常使用的数据结构是位图（Bitmap），其包含一张图片所有数据。整个图片由点阵和颜色值组成，点阵是一个包含像素的矩阵，每个元素对应图片一个像素。颜色值（ARGB），分别对应透明度、红、绿、蓝四个通道分量，其共同决定每个像素点显示的颜色。

**（1）色彩矩阵分析**

> 在色彩处理中，通常使用以下三个角度来描述一个图像。

- 色调 —— 物体传播的颜色
- 饱和度 —— 颜色的纯度，从 0（灰）到 100%（饱和）来进行描述
- 亮度 —— 颜色的相对明暗程度

在Android中，系统使用一个颜色矩阵 —— ColorMatrix 来处理图像的色彩效果。Android 中的颜色矩阵是一个4×5 的数字矩阵，用来对图片的色彩进行处理。对于每个像素点，都有一个颜色分量矩阵用来保存颜色的RGBA值。图中矩阵A是一个 4×5 的颜色矩阵，在Android 中，它会以一维数组的形式来存储 [a,b,c,d,e,f,g,h,i,j,k,I,m,n,o,p,q,r,s,t] ，C 是颜色矩阵分量。在处理图像时，使用矩阵乘法运算 AC 来处理颜色分量矩阵。这个矩阵通常被用来作为初始的颜色矩阵来使用，它不会对原有颜色值进行任何变化。

<img src="Image.assets\颜色处理.jpg" alt="颜色处理" style="zoom:67%;" />

<img src="Image.assets\颜色矩阵.jpg" alt="颜色矩阵" style="zoom:67%;" />

> 从这个公式可以发现，对于 4×5 颜色矩阵按以下方式划分：

- 第一行的 abcde 值用来决定新的颜色值中的R —— 红色
- 第二行的 fghij 值用来决定新的颜色值中的G —— 绿色
- 第三行的 klmno 值用来决定新的颜色值中的B —— 蓝色
- 第四行的 pqrst 值用来决定新的颜色值中的A —— 透明度
- 矩阵 A 的第五列 —— ejot 值分别用来决定每个分量中的 offset（偏移量）

> 当变换颜色值时，通常有两种方法：

- 一是直接改变颜色的 offset（偏移量）的值来修改颜色分量
- 另一个是直接改变对应 RGBA 值的系数来调整颜色分量的值

i、改变偏移量

<img src="Image.assets\颜色偏移量.jpg" alt="颜色偏移量" style="zoom:67%;" />

在上面矩阵中，修改R、G对应的颜色偏移量，最后的处理结果是图像的红、绿色分量增加了100。而红绿混合会得到黄色，最终是让整个图像的色调偏黄色

ii、改变颜色系数

<img src="Image.assets\颜色系数.jpg" alt="颜色系数" style="zoom:67%;" />

在上面矩阵中，改变 G 分量对应系数 g，在矩阵运算后 G 分量变为以前的两倍，效果是图像的色调偏绿

iii、改变色光属性

色调、饱和度、亮度这三个属性在图像处理中经常使用。在颜色矩阵中也封装了一些 API 来快速调整这些参数。

在Android中，系统封装一个类 —— ColorMatrix（颜色矩阵）。通过该类，可以方便地通过改变矩阵值来处理颜色效果。

> 创建一个ColorMatrix对象非常简单。

 `ColorMatrix colorMatrix = new ColorMatrix();` 

> 处理不同的色光属性

- 色调：Android系统提供 `setRotate(int axis,float degree)` 帮助设置颜色色调。第一个参数，系统分别使用0、1、2来代表Red、Green、Blue三种颜色；第二个参数，是需要处理的值。该方法可以为 RGB 三种颜色分量分别重新设置不同的色调值。

  ```java
  ColorMatrix hueMatrix = new ColorMatrix();
  hueMatrix.setRotate(0, hueO);
  hueMatrix.setRotate(1, hue1);
  hueMatrix.setRotate(2, hue2);
  ```

- 饱和度：Android系统提供 `setSaturation(float sat)` 方法来设置颜色饱和度，参数代表设置颜色饱和度的值，当饱和度为 0 时，图像变成灰度图像。

  ```java
  ColorMatrix saturationMatrix = new ColorMatrix();
  saturationMatrix.setSaturation(saturation);
  ```

- 亮度：当三原色以相同比例混合时，会显示白色。系统也使用该原理来改变图像亮度。当亮度为 0 时，图像变为全黑。

  ```java
  ColorMatrix lumMatrix = new ColorMatrix();
  lumMatrix.setScale(lum,lum,lum,1);
  ```

- 除了上面三种方式进行颜色效果的处理外，Android 系统还封装了矩阵乘法运算。提供 `postConcat()` 方法将矩阵的作用效果混合，从而叠加处理效果。

  ```java
  ColorMatrix imageMatrix = new ColorMatrix();
  imageMatrix.postConcat(hueMatrix);
  imageMatrix.postConcat(saturationMatrix);
  imageMatrix.postConcat(lumMatrix);
  ```

在设置好处理的颜色矩阵后，通过使用 Paint 类的 `setColorFilter()` 方法，将通过 imageMatrix 构造的 ColorMatrixColorFilter 对象传递进去，并使用该画笔绘制原来的图像，从而将颜色矩阵作用到原图中。

Android 系统不允许直接修改原图，类似 Photoshop 中的锁定，必须通过原图创建一个同样大小的 Bitmap，将原图绘制到该 Bitmap 中，以副本形式来修改。代码如下，bm即为原图，bmp为创建的副本。

```java
Bitmap bmp = Bitmap.createBitmap(bm.getWidth(), bm.getHeight(),Bitmap.Config.ARGB_8888);
Canvas canvas = new Canvas(bmp);
Paint paint = new Paint();
canvas.drawBitmap(bm,0,0,paint);
```

**（2）Android颜色矩阵 —— ColorMatrix**

模拟一个颜色矩阵

![颜色矩阵模拟](Image.assets\颜色矩阵模拟.png)

```xml
<GridLayout
    android:id="@+id/group"
    android:layout_width="match_parent"
    android:layout height="0dp"
    android:layout_weight="3"
    android:columnCount="5"
    android:rowCount"4">
</GridLayout>
```

> 动态创建EditText

```java
mGroup = (GridLayout)findViewByld(R.id.group);

mGroup.post(new Runnable(){
    @Override
    public void run(){
        //获取宽高信息
        mEtWidth = mGroup.getWidth() / 5;
        mEtHeight = mGroup.getHeight() / 4;
        addEts();
    }
});

//添加EditText
private void addEts(){
    for (int i=0;i<20;i++){
        EditText editText = new EditText(this);
        mEts[i] = editText;
        mGroup.addView(editText,mEtWidth,mEtHeight);
    }
}
```

无法在 `onCreate()` 方法中获得视图的宽高值，所以通过 View 的 post 方法在视图创建完毕后获得其宽高值。

> 将矩阵设置到图像

```java
//获取矩阵值
private void getMatrix(){
    for(int i=0;i<20;i++){
    	mColorMatrix[i] = Float.valueOf(mEts[i].getText().toString());
    }
}
                                              
//将矩阵值设置到图像
private void setImageMatrix(){
    Bitmap bmp = Bitmap.createBitmap(
                bitmap.getWidth(),
                bitmap.getHeight(),
                Bitmap.Config.ARGB_8888);
    android.graphics.ColorMatrix colorMatrix = new android.graphics.ColorMatrix();
    colorMatrix.set(mColorMatrix);
    Canvas canvas = new Canvas(bmp);
    Paint paint = new Paint();
    paint.setColorFilter(new ColorMatrixColorFilter(colorMatrix));
    canvas.drawBitmap(bitmap,0,0,paint);
    mImageView.setImageBitmap(bmp);
}
```

**（3）常用图像颜色矩阵处理效果**

i、灰度效果

> 颜色矩阵

```
0.33F, 0.59F, 0.11F, 0, 0,
0.33F, 0.59F, 0.11F, 0, 0,
0.33F, 0.59F, 0.11F, 0, 0,
0	 , 0    , 0	   , 1, 0,
```

ii、图像反转

> 颜色矩阵

```
-1, 0,  0, 1, 0,
 0, 1,  0, 1, 1,
 0, 0, -1, 1, 1
 0, 0,  0, 1, 0，
```

![颜色反转](Image.assets\颜色反转.png)

iii、怀旧效果

> 颜色矩阵

```
0.393F, 0.769F, 0.189F, 0, 0,
0.349F, 0.686F, 0.168F, 0, 0,
0.272F, 0.534F, 0.131F, 0, 0,
	 0,		 0,	     0, 1, 0,
```

iv、去色效果

> 颜色矩阵

```
1.5F, 1.5F, 1.5F, 0, -1,
1.5F, 1.5F, 1.5F, 0, -1,
1.5F, 1.5F, 1.5F, 0, -1,
   0,    0,    0, 0,  1,
```

v、高饱和度

> 颜色矩阵

```
 1.438F, -0.122F, -0.016F, 0, -0.03F,
-0.062F,  1.378F, -0.016F, 0,  0.05F,
-0.062F, -0.122F,  1.483F, 0, -0.02F,
	  0,	   0,	    0, 1, 	   0,
```

**（4）像素点分析**

更精确的图像处理方式，通过改变每个像素点具体 ARGB 值来处理图像效果。

传递进来的原始图片不能修改（mutable），一般根据原始图片生成一张新的图片来修改。

Android 系统提供 `Bitmap.getPixels()` 方法提取整个 Bitmap 中的像素点，并保存到一个数组中。

 `bitmap.getPixels(pixels,offset,stride,x,y,width,height)` 

- pixels：接收位图颜色值的数组
- offset：写入 pixels[] 中的第一个像素索引值
- stride： pixels[] 中的行间距
- x：从位图中读取的第一个像素的 x 坐标值
- y：从位图中读取的第一个像素的 y 坐标值
- width：从每一行中读取的像素宽度
- height：读取的行数

> 通常情况下，可以使用：

 `bitmap.getPixels(oldPx,0,bm.getWidth(),0,0,width,height);` 

> 获取每个像素具体 ARGB 值

```java
color = oldPx[i];
r = Color.red(color);
g = Color.green(color);
b = Color.blue(color);
a = Color.alpha(color);
```

> 重构图像

```java
rl = (int)(0.393 * r + 0.769 * g + 0.189 * b);
gl = (int)(0.349 * r + 0.686 * g + 0.168 * b);
bl = (int)(0.272 * r + 0.534 * g + 0.131 * b);
//将新的RGBA值合成像素点。
newPx[i] = Color.argb(a,rl,gl,b1);
//将处理之后的像素点数组重新set给Bitmap，实现图像处理。
bmp.setPixels(newPx,0,width,0,0,width,height);
```

**（5）常用图像像素点处理效果**

i、底片效果

```java
public static Bitmap handlelmageNegative(Bitmap bm){
    int width = bm.getWidth();
    int height = bm.getHeight();
    int color;
    int r,g,b,a;
    
    Bitmap bmp = Bitmap.createBitmap(width,height,Bitmap.Config.ARGB_8888);
 
    int[] oldPx = new int[width * height];
    int[] newPx = new int[width * height];
    bm.getPixels(oldPx,0,width,0,0,width,height);
    for(int i=0;i<width * height;i++){
        color = oldPx[i];
        r = Color.red(color);
        g = Color.green(color);
        b = Color.blue(color);
        a = Color.alpha(color);
        
        //底片效果
        r = 255 - r;
        g = 255 - g;
        b = 255 - b;
        
        if(r > 255){
        	r = 255;
        }else if(r < 0){
        	r = 0;
        }
        
        if(g > 255){
        	g = 255;
        }else if(g < 0){
        	g = 0;
        }
        
        if(b > 255){
        	b = 255;
        }else if(b < 0){
        	b = 0;
        }
        
        newPx[i] = Color.argb(a,r,g,b);
    }
    bmp.setPixels(newPx,0,width,0,0,width,height);
    return bmp;
}
```

ii、老照片效果

```java
r1 = (int)(0.393 * r + 0.769 * g + 0.189 * b);
g1 = (int)(0.349 * r + 0.686 * g + 0.168 * b);
b1 = (int)(0.272 * r + 0.534 * g + 0.131 * b);
```

iii、浮雕效果

> 若存在 ABC 3个像素点，B 点对应的浮雕效果算法：

```java
B.r = C.r - B.r + 127;
B.g = C.g - B.g + 127;
B.b = C.b - B.b + 127;
```

#### 6、Android图像处理之图形特效处理

**（1）Android变形矩阵Matrix**

对图像的图形变换，Android 系统通过矩阵处理，每个像素点表达其坐标的X、Y信息。Android 的图形变换矩阵是一个 3×3 的矩阵。

![图形矩阵](Image.assets\图形矩阵.png)

```
X1 = a * X + b * Y + c
Y1 = d * X + e * Y + f
l = g * X + h * Y + i
```

通常情况下，会让 g = h = 0，i = l，这样使 1= g * X + h * Y + i 恒成立。

图形变换矩阵有一个初始矩阵，是对角线元素 a、e、i 为1，其他元素为0的矩阵。

![图形初始矩阵](Image.assets\图形初始矩阵.png)

> 图像的变形处理通常包含以下四类基本变换

- Translate：平移变换
- Rotatc：旋转变换
- Scale：缩放变换
- Skew：错切变换

i、平移变换

平移变换即将每个像素点都进行平移变换。当从 p(x0,y0) 平移到 p(x,y) 时，坐标值发生了如下所示的变换。

```
X = X0 + △X
Y = Y0 + △y
```

![平移变换](Image.assets\平移变换.png)

ii、旋转变换

旋转变换即指一个点围绕一个中心旋转到一个新的点

当从 p(x0,y0) 点以坐标原点为旋转中心旋转到 p(x,y) 点时，可将点的坐标都表达成 OPX 轴正方向夹角的函数表达式

```
X0 = r·cosα
y0 = r·sinα
x = r·cos(α+θ) = r·cosα·cosθ - r·sinα·sinθ = x0cosθ - y0sinθ
y = r·sin(α+θ) = r·sinα·cosθ + r·cosα·sinθ = y0cosθ + x0sinθ
```

![旋转变换](Image.assets\旋转变换.png)

> 以任意点 О 为旋转中心进行旋转变换通常需要以下三个步骤：

- 将坐标原点平移到О点。
- 使用前面以坐标原点为中心的旋转方法进行旋转变换。
- 将坐标原点还原。

iii、缩放变换

一个像素点不存在缩放概念，但图像由很多个像素点组成，如果将每个点的坐标进行相同比例的缩放，就会形成整个图像缩放的效果。

```
x = K1 * x0
y = K2 * y0
```

![缩放变换](Image.assets\缩放变换.png)

vi、错切变换

错切变换（skew）在数学上又称为 Shear mapping （”剪切变换”）或者 Transvection（缩并）。错切变换的效果是让所有点的 X 坐标（或 y 坐标）保持不变，而对应的 y 坐标（或 x 坐标）按比例发生平移，平移大小和该点到x轴（或y轴）的垂直距离成正比。错切变换通常包含两种：水平错切与垂直错切。

<img src="Image.assets\水平垂直错切.png" alt="水平垂直错切" style="zoom:67%;" />

```
x = x0 + K1 * y0
y = K2 * x0 + y0
```

![错切变换矩阵](Image.assets\错切变换矩阵.png)

- A、E 控制 Scale：缩放变换
- B、D 控制 Skew：错切变换
- C、F 控制 Trans：平移变换
- A、B、D、E 共同控制 Rotate：旋转变换

在图形变换矩阵中，通过一个一维数组模拟矩阵，通过 `setValues()` 方法将一个一维数组转换为图形变换矩阵

```java
private float[] mImageMatrix = new float[9];
Matrix matrix = new Matrix();
matrix.setValues(mImageMatrix);
```

获得变换矩阵后，可将一个图像以该变换矩阵的形式绘制出来

  `canvas.drawBitmap(mBitmap,matrix,null);` 

> Android 使用 Matri 类来封装矩阵，并提供操作方法实现上面四种变换方式

-  matri X .setRotate()：旋转变换
- matri X .setTranslate()：平移变换
- matri X .setScale()：缩放变换
- matri X .setSkew()：错切变换
-  `pre()` 和 `post()` ：提供矩阵的前乘和后乘运算

Matri 类的 set 方法会重置矩阵中的所有值，而 post 和 pre 方法不会，这两个方法常用来实现矩阵的混合作用。

矩阵运算不满足交换率，矩阵乘法的前乘和后乘是两种不同的运算方式。

**（2）像素块分析**

 `drawBitmapMesh()` 把图像分成一个个小块，通过改变每个图像块来修改整个图像。

 `drawBitmapMesh(Bitmap bitmap,int meshWidth,int meshHeight,float[] verts,int vertOffset,int[] colors,int colorOffset,Paint paint)`

- bitmap：将要扭曲的图像
- meshWidth：需要的横向网格数目
- meshHeight：需要的纵向网格数目
- verts：网格交叉点坐标数组
- vertOffset：verts 数组中开始跳过的 (x,y) 坐标对的数目

> verts

在图像上横纵各画 N-l 条线，将图像分成 N 块，这横纵各 N 条线就交织成了 N×N 个点，每个点的坐标以xi，yi的形式保存在 verts 数组中。verts 数组的每两位用来保存一个交织点，第一个是横坐标，第二个是纵坐标。 `drawBitmapMesh()` 方法靠这些坐标值的改变来重新定位每一个图像块，达到图像效果处理。

当改变某些交叉点的坐标时，就会让整个像素块随之发生扭曲。 `drawBitmapMesh()` 方法功能非常强大，基本上可以实现所有的图像特效，使用起来也非常复杂，关键在于计算、确定新交叉点的坐标。

#### 7、Android图像处理之画笔特效处理

**（1）PorterDuffXfermode**

PorterDuffXfermode设置的是两个图层交集区域的显示方式，dst是先画的图形，而src是后画的图形。

用的最多的是使用一张图片作为另一张图片的遮罩层，通过控制遮罩层的图形，来控制下面被遮罩图形的显示效果。其中最常用的就是通过DST_IN、SRC_IN模式实现将一个矩形图片变成圆角图片或者圆形图片的效果。PorterDuffXfermode 进行图层混合时并不只进行图层的计算，同时也会计算透明通道的值。

<img src="Image.assets\PorterDuffXfermode.png" alt="PorterDuffXfermode" style="zoom:67%;" />

在使用 PorterDuffXfermode 时最好将硬件加速关闭，因为有些模式不支持硬件加速。

**（2）Shader**

Shader 又被称为着色器、渲染器，它用来实现一系列的渐变、渲染效果。Android 中的Shader包括以下几种。

- BitmapShader：位图Shader
- LinearGradient：线性Shader
- RadialGradient：光束Shader
- SweepGradient：梯度Shader
- ComposeShader：混合Shader

第一个 Shader 与其他 Shader 产生的渐变不同，BitmapShader 产生的是一个图像，有点像 Photoshop
中的图像填充渐变。作用是通过 Paint 对画布进行指定 Bitmap 的填充，填充时有以下几种模式可以选择。

- CLAMP拉伸：拉伸的是图片最后的那一个像素，不断重复（将图像设置为一定大小，就可避免这种拉伸）
- REPEAT重复：横向、纵向不断重复
- MIRROR镜像：横向不断翻转重复，纵向不断翻转重复

> 将图片垂直翻转

```java
Matrix matrix = new Matrix();
matrix.setScale(1F.-1F);
mRefBitmap = Bitmap.createBitmap(mSrcBitmap,0,0,mSrcBitmap.getWidth(),mSrcBitmap.getHeight(),matrix,true);
```

同理可实现水平翻转

**（3）PathEffect**

PathEffect 指用各种笔触效果来绘制一个路径。Android 系统提供如图展示的几种绘制 PathEffect 的方式，从上到下依次是：

- 没效果
- CornerPathEffect：将拐角处变得圆滑，具体圆滑程度由参数决定
- DiscretePathEffect：线段上会产生许多杂点
- DashPathEffect：该效果可用来绘制虚线，用一个数组设置各点之间间隔，此后绘制虚线时重复该间隔进行绘制，另一个参数 phase 用来控制绘制时数组的一个偏移量，可通过设置值来实现路径的动态效果
- PathDashPathEffect：与上一个类似，但功能更强大，可设置显示点的图形
- ComposePathEffect：该方法的功能是将任意两种路径特性组合形成新效果

<img src="Image.assets\PathEffect.png" alt="PathEffect" style="zoom:80%;" />

#### 8、SurfaceView

**（1）SurfaceView 与View的区别**

Android 系统提供 View 进行绘图处理，View 通过刷新来重绘视图，Android 系统通过发出 VSYNC 信号来进行屏幕的重绘，刷新的间隔时间为16ms。如果在16ms 内 View 完成所需执行的所有操作，那么用户在视觉上就不会产生卡顿感觉，而如果执行的操作逻辑太多，特别是需要频繁刷新的界面上，例如游戏界面，就会不断阻塞主线程，从而导致画面卡顿。很多时候，在自定义 View 的 Log 中经常会看见如下警告：

"Skipped 47 frames! The application may be doing too much work on its main thread”

这些警告的产生，很多情况下是因为在绘制过程中逻辑太多造成的。

SurfaceView 与 View 有所不同的，区别主要体现在：

- View 主要适用于主动更新的情况下，SurfaceView 主要适用于被动更新，例如频繁地刷新
- View 在主线程中对画面进行刷新，SurfaceView 通常会通过一个子线程来进行页面的刷新
- View 在绘图时没有使用双缓冲机制，SurfaceView 在底层实现机制中就已经实现了双缓冲机制

如果自定义View需要频繁刷新，或者刷新时数据处理量比较大，可以考虑使用SurfaceView来取代View 。

**（2）SurfaceView的使用**

大部分 SurfaceView 绘图操作可套用以下模板代码进行编写。

>  创建一个SurfaceView的模板

- 创建SurfaceView

  - 创建自定义 SurfaceView 继承自 SurfaceView，实现两个接口（SurfaceHolder.Callback 和 Runnable）

  ```java
  public class SurfaceViewTemplate extends SurfaceView implements SurfaceHolder.Callback,Runnable
  ```

  - 在自定义 SurfaceView 中实现接口方法

    - 对于 SurfaceHolder.Callback 方法

    ```java
    @Override
    public void surfaceCreated(SurfaceHolder holder){
    }
    @Override
    public void surfaceChanged(SurfaceHolder holder,int format, int width,int height){
    }
    @Override
    public void surfaceDestroyed(SurfaceHolder holder){
    }
    ```

    分别对应 SurfaceView 的创建、改变和销毁过程。

    - 对于Runnable接口，需要实现 `run()` 方法

    ```java
    @Override
    public void run(){
    }
    ```

- 初始化 SurfaceView

在自定义 SurfaceView 的构造方法中，需要对 SurfaceView 初始化。在自定义 SurfaceView中，通常需要定义以下三个成员变量：

```java
//SurfaceHolder
private SurfaceHolder mHolder;
//用于绘图的Canvas
private Canvas mCanvas;
//子线程标志位
private boolean mIsDrawing;
```

初始化方法初始化一个 SurfaceHolder 对象，并注册 SurfaceHolder 的回调方法。

```java
mHolder = getHolder();
mHolder.addCallback(this);
```

Canvas 与在 View 的 `onDraw()` 方法中使用 Canvas 绘图一样，在SurfaceView中也使用 Canvas 进行绘图

标志位用来控制子线程的，SurfaceView 通常会起一个子线程来进行绘制，该标志位可以控制子线程

- 使用SurfaceView

SurfaceHolder 对象的 `lockCanvas()` 方法可以获得当前 Canvas 绘图对象，获取到的 Canvas 对象还是继续上次的 Canvas 对象，之前的绘图操作都将被保留，如需擦除，可在绘制前通过 `drawColor()` 方法进行清屏

绘制时利用 SurfaceView 三个回调方法在 `surfaceCreated()` 方法中开启子线程进行绘制，子线程使用一个 `while(mIsDrawing)` 循环不停进行绘制，在绘制的具体逻辑中通过 `lockCanvas()` 方法获得的 Canvas 对象进行绘制，通过 `unlockCanvasAndPost(mCanvas)` 方法对画布内容进行提交。

```java
package com.ziya.surfaceviewtest;

import android.content.Context;
import android.graphics.Canvas;
import android.util.AttributeSet;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

public class SurfaceViewTemplate extends SurfaceView implements SurfaceHolder.Callback,Runnable{
    //SurfaceHolder
    private SurfaceHolder mHolder;
    //用于绘图的Canvas
    private Canvas mCanvas;
    //子线程标志位
    private boolean mIsDrawing;
    
    public SurfaceViewTemplate(Context context){
        super(context);
        initView();
    }
    
    public SurfaceViewTemplate(Context context,AttributeSet attrs){
        supcr(context,attrs);
        initView();
    }
    
    public SurfaceViewTemplate(Context context,AttributeSet attrs,int defStyle){
        super(context,attrs,defStyle);
        initView();
    }
    
    private void initView(){
        mHolder = getHolder();
        mHolder.addCallback(this);
        //获取焦点，true才能获取控件点击事件
        setFocusable(true);//对应键盘操作
        setFocusablelnTouchMode(true);//对应触屏操作
        this.setKeepScreenOn(true);//设置不息屏
        //mHolder.setFormat(PixelFormat.OPAQUE);//android默认颜色格式，不带Alpha值
    }
    
    @Override
    public void surfaceCreated(SurfaceHolder holder){
        mIsDrawing = true;
        new Thread(this).start();
    }
    
    @Override
    public void surfaceChanged(SurfaceHolder holdcr,int format,int width,int height){
    }
    
    @Override
    public void surfaceDestroyed(SurfaceHolder holder){
        mIsDrawing = false;
    }
    
    @Override
    public void run(){
        while(mIsDrawing){
        	draw();
        }
    }
    
    private void draw(){
        try {
            mCanvas = mHolder.lockCanvas();
            //draw something
        }catch (Exception e){
        }finally{
        	if(mCanvas != null){
                //放在finally块中保证每次都能提交内容
        		mHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
}
```

**（3）SurfaceView 实例**

i、正弦曲线

在界面上不断绘制一个正弦曲线，类似示波器、心电图、股票走势图等

要绘制一个正弦曲线，只需不断修改横纵坐标值，并让其满足正弦函数

使用一个 Path 对象保存正弦函数上的坐标点，在子线程的 while 循环中，不断改变横纵坐标值

```java
@Override
public void run(){
    while(mIsDrawing){
        draw();
        x += 1;
        y = (int)(100 * Math.sin(x * 2 * Math.PI / 180) + 400);
        mPath.lineTo(x,y);
    }
    
private void draw(){
    try{
        mCanvas = mHolder.lockCanvas();
        //SurfaceView 背景
        mCanvas.drawColor(Color.WHITE);
        mCanvas.drawPath(mPath,mPaint);
    }catch(Exception e){
    }finally{
        if(mCanvas != null){
        	mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```

ii、绘图板

用 SurfaceView 实现一个简单绘图板，绘图方法与在 View 中绘图所用方法一样，通过 Path 对象记录手指滑动路径进行绘图，在 SurfaceView 的 `onTouchEvent()` 中记录Path路径

```java
@Override
public boolean onTouchEvent(MotionEvent event){
    int x = (int)event.getX();
    int y = (int)event.getY();
    switch(event.getAction()){
        case MotionEvent.ACTION_DOWN:
        	mPath.moveTo(x,y);break;
        case MotionEvent.ACTION_MOVE:
        	mPath.lineTo(x,y);break;
        case MotionEvent.ACTION_UP:
        	break;
    }
    return true;
}
```

在 `draw()` 方法中绘制

```java
private void draw(){
    try {
        mCanvas = mHolder.lockCanvas();
        mCanvas.drawColor(Color.WHITE);
        mCanvas.drawPath(mPath,mPaint);
    }catch (Exception e){
    } finally {
        if(mCanvas != null){
        	mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```

在子线程循环中优化。在线程中不断调用 `draw()` 方法进行绘制，但有时绘制不用这么频繁，可在子线程中进行 sleep 操作，尽可能节省系统资源

```java
@Override
public void run(){
    long start = System.currentTimeMillis():
    while (mIsDrawing){
    	draw();
    }
    long end = System.currentTimeMillis();
    //50 - 100
    if(end - start < 100){
        try {
        	Thread.sleep(100 - (end - start));
        } catch (InterruptedException e){
        	e.printStackTrace();
        }
    }
}
```

判断 `draw()` 方法所用逻辑时长确定 sleep 时长，是一个非常通用的解决方案，100ms 是一个大致的经验值，其取值一般在 50ms 到 100ms 左右

### 七、Android动画机制与使用技巧

#### 1、Android View动画框架

Animation 框架定义了透明度、旋转、缩放和位移几种常见的动画，且控制的是整个View，实现原理是在每次绘制视图时 View 所在 ViewGroup 中的 drawChild 函数获取该 View 的 Animation 的 Transformation 值，然后调用 `canvas.concat(transformToApply.getMatrix())` ，通过矩阵运算完成动画帧。如果动画没完成，就继续调用 `invalidate()` 函数，启动下次绘制来驱动动画，从而完成整个动画的绘制。

视图动画提供了 AlphaAnimation、RotateAnimation、TranslateAnimation、ScaleAnimation 四种动画方式，提供了 AnimationSet 动画集合、混合使用多种动画。

Android 3.0 之前，视图动画一家独大，随着 Android 3.0 之后属性动画框架的推出，其大不如前。

相比属性动画，视图动画的一个非常大的缺陷是不具备交互性，某个元素发生视图动画后，其响应事件的位置依然在动画前的地方，视图动画只能做普通的动画效果，避免交互发生。

其优点是效率比较高且使用方便。视图动画使用简单，可通过 XML 文件描述动画过程，也可用代码控制整个动画过程

**（1）透明度动画**

> 为视图增加透明度变换动画

```java
AlphaAnimation aa = new AlphaAnimation(0,1);//起始透明度和终止透明度，1为不透明，0为透明
aa.setDuration(1000);//设置持续时间
view.startAnimation(aa);
```

**（2）旋转动画**

> 为视图增加旋转变换动画

```java
RotateAnimation ra = new RotateAnimation(0,360,100,100);//参数是旋转起始角度和旋转中心点坐标，分别是与初始点(0,0)的X、Y轴的差值
ra.setDuration(1000);
view.startAnimation(ra);
```

> 设置旋转参考系为自身中心点

```java
RotateAnimation ra = new RotateAnimation(0,360,
                    RotateAnimation.RELATIVE_TO_SELF,0.5F,
                    RotateAnimation.RELATIVE_TO_SELF,0.5F);
```

**（3）位移动画**

> 为视图增加位移变换动画

```java
TranslateAnimation ta = new TranslateAnimation(0,200,0,300);//参数是开始位置和终止位置，分别是与初始点(0,0)的X、Y轴的差值
ta.setDuration(1000),
view.startAnimation(ta);
```

**（4）缩放动画**

> 为视图的缩放增加动画效果

```java
ScaleAnimation sa = new ScaleAnimation(0,2,0,2);//参数分别是起始与终止时X轴和Y轴的伸缩尺度，分别是与初始点(0,0)的X、Y轴的差值
sa.setDuration(1000);
view.startAnimation(sa);
```

> 设置缩放中心点为自身中心

```java
ScalcAnimation sa = new ScaleAnimation(0,1,0,1,
                    Animation.RELATIVE_TO_SELF,0.5F,
                    Animation.RELATIVE_TO_SELF,0.5F);
sa.setDuration(1000);
view.startAnimation(sa);
```

**（5）动画集合**

> 通过 AnimationSet 可以将动画以组合的形式展现出来

```java
AnimationSet as = new AnimationSet(true);
as.setDuration(1000);
AlphaAnimation aa = new AlphaAnimation(0,1);
aa.setDuration(1000);
as.addAnimation(aa);
TranslateAnimation ta = new TranslatcAnimation(0,100,0,200);
ta.setDuration(1000);
as.addAnimation(ta);
view.startAnimation(as);
```

对于动画事件，Android 提供了对应的监听回调，要添加相应的监听方法

```java
animation.setAnimationListener(new Animation.AnimationListener(){
    @Override
    public void onAnimationStart(Animation animation){
    }
    @Override
    public void onAnimationEnd(Animation animation){
    }
    @Override
    public void onAnimationRepeat(Animation animation){
    }
});
```

通过监听回调可以获取到动画的开始、结束和重复事件

#### 2、Android 属性动画分析

在 Animator 框架中使用最多的是 AnimatorSet 和 ObjectAnimator 配合，使用 ObjectAnimator 进行更精细化控制，只控制一个对象的一个属性值，使用多个 ObjectAnimator 组合到 AnimatorSet 形成一个动画，且ObjectAnimator 能自动驱动，调用 `setFrameDelay(longframeDelay)` 设置动画帧之间的间隙时间，调整帧率,减少动画过程中频繁绘制界面，在不影响动画效果的前提下减少 CPU 资源消耗。

属性动画通过调用属性的 get、set 方法来控制一个View的属性值，基本可实现所有动画效果

**（1）ObjectAnimator**

ObjectAnimator 是属性动画框架中最重要的实行类

创建一个 ObjectAnimator 只需通过其静态工厂类直接返回一个 ObjectAnimator 对象。参数包括一个对象和对象的属性名字，该属性必须有 get 和 set 函数，内部会通过 Java 反射机制调用 set 函数修改对象属性值。也可调用setInterpolator 设置相应的差值器

```java
ObjectAnimator animator = ObjectAnimator.ofFloat(
                            view,
                            "translationX",
                            300);
animator.setDuration(300);
animator.start();
```

通过 ObjectAnimator 的静态工厂方法，创建一个 ObjectAnimator 对象。第一个参数是要操纵的 View，第二个参数是要操纵的属性，最后一个参数是一个可变数组参数，需要传进去该属性变化的一个取值过程，这里只设置了一个参数，即变化到 300。与视图动画一样，也可以给属性动画设置显示时长、差值器等属性，与在视图动画中的设置方法类似

在使用 ObjectAnimator 时，要操纵的属性必须有 get、set 方法，不然 ObjectAnimator 无法起效。下面是一些常用的可以直接使用属性动画的属性值：

- translationX 和 translationY：这两个属性作为增量控制 View 对象从布局容器左上角坐标偏移的位置
- rotation、rotationX 和 rotationY：这三个属性控制 View 对象围绕支点进行 2D 和 3D 旋转
- scaleX 和 scaleY：这两个属性控制 View 对象围绕支点 2D 缩放
- pivotX 和 pivotY：这两个属性控制 View 对象支点位置，围绕该支点旋转和缩放变换处理。默认该支点位置是View 对象的中心点
- x 和 y：这两个属性描述 View 对象在容器中的最终位置，是最初左上角坐标和 translationX、translationY 值的累计和
- alpha：表示 View 对象的 alpha 透明度。默认是1（不透明），0 代表完全透明（不可见）

若一个属性没有 get、set 方法，Google 在应用层提供两种方案来解决，一是通过自定义属性类或包装类，来间接给该属性增加 get、set 方法，或通过 ValucAnimator 实现

> 使用包装类方法给属性值增加 get、set 方法

```java
private static class WrapperView {
    private View mTarget;
    
    public WrapperView(View target){
        mTarget = target;
    }
    public int getWidth(){
        return mTarget.getLayoutParams().width;
    }
    public void setWidth(int width){
        mTarget.getLayoutParams().width = width;
        mTarget.requestLayout();
    }
}
```

使用时只需操作包装类就可间接调用 get、set 方法

```java
ViewWrapper wrapper = new ViewWrapper(mButton);
ObjectAnimator.ofInt(wrapper,"width",500).setDuration(5000).start();
```

**（2）PropertyValuesHolder**

在属性动画中，若针对同一对象的多个属性，要同时作用多种动画，可使用 PropertyValuesHolder 实现。比如平移过程中，改变 X、Y 轴的缩放

```java
PropertyValuesHolder pvh1 = PropertyValuesHolder.ofFloat("translationX",300f);
PropertyValuesHolder pvh2 = PropertyValuesHolder.ofFloat("scaleX",1f,0,1f);
PropertyValuesHolder pvh3 = PropertyValuesHolder.ofFloat("scaleY",1f,0,1f);
ObjectAnimator.ofPropertyValuesHolder(view, phl, pvh2, pvh3).setDuration(1000).start();
```

分别使用 PropertyValuesHolder 对象控制 translationX、scaleX、scaleY 三个属性，调用ObjectAnimator.ofPropertyValucsHolder 方法实现多属性动画的共同作用，类似AnimationSet的使用

**（3）ValueAnimator**

ObjectAnimator 继承自 ValueAnimator
 `public final class ObjectAnimator extends ValueAnimator` 

ValueAnimator 不提供动画效果，像一个数值发生器，用来产生具有一定规律的数字，让调用者控制动画的实现过程，通常情况下在 ValueAnimator 的 AnimatorUpdatcListener 中监听数值的变换，完成动画变换

```java
ValueAnimator animator = ValueAnimator.ofFloat(0,100);
animator.setTarget(view);
animator.setDuration(1000).start();
animator.addUpdateListener(new AnimatorUpdateListener(){
    @Override
    public void onAnimationUpdate(ValueAnimator animation){
        Float value = (Float) animation.getAnimatedValue();
        // TODO use the value
    }
});
```

**（4）动画事件的监听**

一个完整的动画具有 Start、Repeat、End、Cancel 四个过程，通过 Android 提供接口，可以方便监听到这四个事件

```java
ObjectAnimator anim = ObjectAnimator.ofFloat(view,"alpha",0.5f);
anim.addListener(new AnimatorListener(){
    @Override
    public void onAnimationStart(Animator animation){
    }
    @Override
    public void onAnimationRepeat(Animator animation){
    }
    @Override
    public void onAnimationEnd(Animator animation){
    }
    @override
    public void onAnimationCancel(Animator animation){
    }
});
anim.start();
```

大部分时候只关心 onAnimationEnd 事件，Android 提供一个 AnimatorListenerAdapter 选择必要事件进行监听

```java
anim.addListener(new AnimatorListenerAdapter(){
    @Override
    public void onAnimationEnd(Animator animation){
    }
});
```

**（5）AnimatorSet**

AnimatorSet 不仅能实现一个属性同时作用多个属性动画效果，也能实现更为精确的顺序控制。

```java
ObjectAnimator animatorl = ObjectAnimator.ofFloat(view,"translationX",300f);
ObjectAnimator animator2 = ObjectAnimator.ofFloat(view,"scaleX",1f,0f,1f);
ObjectAnimator animator3 = ObjectAnimator.ofFloat(view,"scaleY".1f,0f,1f);
AnimatorSet set= new AnimatorSet0;
set.setDuration(1000);
set.playTogether(animator1,animator2,animator3);
set.start();
```

在属性动画中，AnimatorSet 通过 `playTogether()` 、`playSequentially()` 、`animSet.play()` 、`with()` 、`before()` 、`after()` 方法控制多个动画协同工作方式，实现对动画播放顺序的精确控制

**（6）在XML中使用属性动画**

```xml
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:propertyName="scaleX"
    android:valueFrom="1.0"
    android:valueTo="2.0"
    android:valueType="floatType">
</objectAnimator>
```

属性动画与视图动画在 XML 文件中写法相似。

在程序中使用 XML 定义的属性动画

```java
public void scaleX(View view){
    Animator anim = AnimatorInflater.loadAnimator(this,R.animator.scalex);
    anim.setTarget(mMv);
    anim.start();
}
```

**（7）View 的 animate 方法**

Android 3.0后 Google 给 View 增加 animate 方法直接驱动属性动画，可认为是属性动画的一种简写方式

```java
view.animate().alpha(0)
    		  .y(300)
              .setDuration(300)
              .withStartAction(new Runnable(){
              		@Override
                	public void run(){
                    }
              })
			  .withEndAction(new Runnable(){
                    @Override
                    public void run{
                        runOnUiThread(new Runnable(){
                            @override
                            public void run(){
                            }
                        });
                    }
              }).start();
```

#### 3、Android 布局动画

布局动画是指作用在 ViewGroup 上，给 ViewGroup 增加 View 时添加一个动画过渡效果。

最简单的布局动画是在 ViewGroup 的 XML 中，使用以下代码打开布局动画：`android:animateLayoutChanges="true"` ，当 ViewGroup 添加 View 时，子 View 有逐渐显示的过渡效果（Android 默认过渡效果，无法使用自定义动画替换）

> 可通过 LayoutAnimationController 类自定义子 View 过渡效果：

```java
LinearLayout ll = (LinearLayout)findViewByld(R.id.ll);
//设置过渡动画
ScaleAnimation sa = new ScaleAnimation(0,1,0,1);
sa.setDuration(2000);
//设置布局动画的显示属性
LayoutAnimationController lac = new LayoutAnimationController(sa,0.5F);
lac.setOrder(LayoutAnimationController.ORDER_NORMAL);
//为ViewGroup设置布局动画
ll.setLayoutAnimation(lac);
```

以上代码给 LinearLayout 增加一个视图动画，让子 View 出现时有一个缩放动画效果

LayoutAnimationController 的第一个参数是需要作用的动画，第二个参数是每个子 View 显示的 delay 时间。当delay 时间不为 0 时，可以设置子 View 显示的顺序

- LayoutAnimationController.ORDER_NORMAL：顺序
- LayoutAnimationController.ORDER_RANDOM：随机
- LayoutAnimationController.ORDER_REVERSE：反序

#### 4、Interpolators（插值器）

通过插值器（Interpolators）可以定义动画变换速率，作用主要是通过控制目标变量的变化值进行对应的变化。同样的动画变换起始值在不同的插值器作用下，每个单位时间内达到的变化值是不一样的。例如一个位移动画，如果使用线性插值器，在持续时间内单位时间移动距离一样，如果使用加速度插值器，单位时间内移动距离越来越快。

#### 5、自定义动画

创建自定义动画需要实现其 applyTransformation 的逻辑即可，通常情况下，还需覆盖父类的 initialize 方法实现初始化工作。

---

applyTransformation 方法有两个参数
 `applyTransformation(float interpolatedTime, Transformation t)` 
第一个参数 interpolatedTime 是插值器的时间因子，是由动画当前完成的百分比和当前时间所对应的插值所计算得来的，取值范围为 0 到 1.0 。
第二个参数 Transformation 是矩阵封装类，一般使用该类获得当前矩阵对象
 `final Matrix matrix = t.getMatrix();` 
通过改变获得的 matrix 对象，可将动画效果实现出来，对于 matrix 的变换操作，基本可实现任何效果的动画

```java
@Override
protected void applyTransformation(float interpolatedTime, Transformation t){
    final Matrix matrix = t.getMatrix();
    //通过matrix的各种操作来实现动画
    matrix.XXXXXX;
}
```

---

> 让一个图片纵向比例不断缩小对应的矩阵处理方法如下：

```java
final Matrix matrix = t.getMatrix();
matrix.preScale(1,
                1 - interpolatedTime,
                mCenterWidth,
                mCenterHeight);
```

mCenterWidth、mCenterHeight 是缩放中心点，设为图片中心。

可设置更精确的插值器，将 0 到 1.0 的时间因子拆分成不同的过程，对不同的过程采用不同的动画效果，模拟更加真实的特效。

---

实例：结合矩阵，使用 Camera 类实现一个自定义 3D 动画效果。Camera 是 android.graphics.Camera 中的Camera 类，它封装了 openGL 的 3D 动画，可以方便地创建 3D 动画效果。类似于一个摄像机，当物体固定在某处时，只要移动摄像机就能拍摄到具有立体感的图像

![camera](Image.assets\camera.png)

```java
@override
public void initialize(int width, int height, int parentWidth, int parentHeight){
    super.initialize(width, height, parentWidth, parentHeight);
    //设置默认时长
    setDuration(2000);
    //动画结束后保留状态
    setFillAfter(true);
    //设置默认插值器
    setlnterpolator(new BounceInterpolator());
    mCenterWidth = width / 2;
    mCenterHeight = height / 2;
}
```

定义动画进行过程

```java
@Override
protected void applyTransformation(float interpolatedTime, Transformation t){
    final Matrix matrix = t.getMatrix();
    mCamera.save();
    //使用Camera设置旋转的角度
    mCamera.rotateY(mRotateY * interpolatedTime);
    //将旋转变换作用到matrix 上
    mCamera.getMatrix(matrix);
    mCamera.restore();
    //通过pre方法设置矩阵作用前的偏移量来改变旋转中心
    matrix.preIranslate(mCenterWidth, mCenterHeight);
    matrix.postTranslate(-mCenterWidth, -mCenterHeight);
}
```

使用 Camera 类实现动画效果无非是设置三个坐标轴的旋转角度，通过最后两行代码改变旋转时的默认旋转中心。

#### 6、Android 5.X SVG 矢量动画机制

Google 在 Android 5.X 中增加对 SVG 矢量图形的支持

> SVG：

- 可伸缩矢量图形（Scalable Vector Graphics）
- 定义用于网络的基于矢量的图形
- 使用 XML 格式定义图形
- 图像在放大或改变尺寸的情况下其图形质量不会有所损失
- 万维网联盟的标准
- 与诸如 DOM 和 XSL 之类的 W3C 标准是一个整体

Bitmap（位图）通过在每个像素点上存储色彩信息来表达图像，SVG 是一个绘图标准。与Bitmap 对比，SVG最大的优点是放大不失真。且 Bitmap 需为不同分辨率设计多套图标，而矢量图不需要

**（1）<path>标签**

使用 <path> 标签创建 SVG，像用指令控制一只画笔，如移动画笔到某一坐标位置，画一条线，结束。

> <path> 标签支持的指令：

- M = moveto(M X,Y)：将画笔移动到指定坐标位置，但未发生绘制
- L = lineto(L X,Y)：画直线到指定坐标位置
- H = horizontal lineto(H X)：画水平线到指定 X 坐标位置
- V = vertical lineto(V Y)：画垂直线到指定 Y 坐标位置
- C = curveto(C X1,Y1,X2,Y2,ENDX,ENDY)：三次贝赛曲线
- S = smooth curveto(S X2,Y2,ENDX,ENDY)：三次贝赛曲线
- Q = quadratic Belzier curve(Q X,Y,ENDX,ENDY)：二次贝赛曲线
- T = smooth quadratic Belzier curveto(T ENDX,ENDY)：映射前面路径后的终点
- A = ellipticalArc(A RX,RY,XROTATION,FLAG1,FLAG2,X,Y)：弧线
- Z = closepath()：关闭路径

> 注意

- 坐标轴以 (0,0) 为中心，X轴水平向右，Y轴水平向下
- 所有指令大小写均可。大写绝对定位，参照全局坐标系；小写相对定位，参照父容器坐标系
- 指令和数据间的空格可省略
- 同一指令出现多次可以只用一个

**（2）SVG常用指令**

- L：绘制直线的指令，代表从当前点绘制直线到给定点。"L” 之后的参数是一个点坐标，如 “L 200 400” 绘制直线。同时可使用 “H” 和 “V” 指令绘制水平、竖直线，后面的参数是 x 坐标( H 指令）或 y 坐标（ V 指令)
- M：M 指令类似 Android 绘图中 path 类的 moveTo 方法，将画笔移动到某一点，但并不发生绘制动作
- A：A 指令用来绘制一段弧线，允许弧线不闭合。可把 A 命令绘制的弧线想象为椭圆某一段，A 指令有七个参数
  - RX，RY：所在椭圆半轴大小
  - XROTATION：椭圆 X 轴与水平方向顺时针方向夹角，可想象为水平椭圆绕中心点顺时针旋转 XROTATION 的角度
  - FLAG1：有两个值，1为大角度弧线，0为小角度弧线
  - FLAG2：有两个值，确定从起点至终点的方向，1为顺时针，0为逆时针
  - X，Y：终点坐标

**（3）SVG编辑器**

SVG 参数的写法固定且复杂，可使用程序实现，一般通过 SVG 编辑器编辑 SVG 图形。网上有很多 SVG 的在线编辑器，通过可视化编辑好图形后，点击 View Source 转换为SVG代码，网址 http:/leditor.method.ac/

离线的 SVG 编辑器，有更多编辑功能，例如 Inkscape

**（4）Android 中使用 SVG**

> Google 在 Android 5.X 中提供两个新的 API 帮助支持SVG：

- VectorDrawable
- AnimatedVectorDrawable

VectorDrawable 可以创建基于 XML 的 SVG 图形，结合 AnimatedVectorDrawable 实现动画效果。

---

i、VectorDrawable

在XML中创建一个静态的 SVG 图形，通常会形成如图树形结构。path 是 SVG 树形结构中的最小单位，通过Group 可将不同的 path 组合

![SVG树形结构](Image.assets\SVG树形结构.png)

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="20Odp"
    android:width="200dp"
    android:viewportHeight="100"
    android:viewportWidth="10O">
</vector>
```

height，width 表示该 SVG 图形的具体大小，viewportHeight，viewportWidth 表示 SVG 图形划分的比例。绘制path 时所使用参数是根据这两个值进行转换的，上面的代码将 200dp 划分为 100 份，如果在绘制图形时使用坐标 (50,50)，意味着该坐标位于该 SVG 图形正中间。

> 给<vector>标签增加显示path

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="200dp"
    android:width="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">
        
    <group
        android:name="test"
        android:rotation="0">
            <path
                android:fillColor="@android:color/holo_blue_light"
                android:pathData="M 25 50 
        						  a 25,25 0 1,0 50,0"/>
    </group>
    
</vector>
```

通过添加 <group> 标签和 <path> 标签绘制 SVG 图形，pathData 是绘制 SVG 图形用到的指令。先使用 M 指令将画笔移动到 (25,50）坐标，再通过 A 指令绘制一个圆孤并填充。使用 android:fillColor 属性绘制出来是一个填充图形，如果要绘制非填充图形，可使用以下属性

```xml
android:strokeColor="@android:color/holo_blue_light"
android:strokeWidth="2"
```

ii、AnimatedVectorDrawable

AnimatedVectorDrawable 的作用是给 VectorDrawable 提供动画效果。首先在 XML 文件中通过 <animated-vector> 标签声明对 AnimatedVectorDrawable 的使用，指定其作用的 path 或 group 

```xml
<animated-vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawablc="@drawable/vector">
    
    <target
            android:name="test"
            android:animation="@anim/anim_path1"/>
                
</animated-vector>
```

对应的 vector 为静态的VectorDrawable

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="200dp"
    android:width="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">
    
    <group
        android:name="test"
        android:rotation="0">
        
        <path
            android:strokeColor="@android:color/holo_blue_light"
            android:strokeWidth="2"
            android:pathData="M 25 50
            				  a 25,25 0 1,0 50,0"/>
                              
    </group>
                              
</vector>
```


AnimatedVectorDrawable 中指定的 target 的 name 属性必须与 VectorDrawable 中需要作用的 name 属性保持—致。通过 AnimatedVectorDrawable 中 target 的 animation 属性将一个动画作用到对应 name 元素上

```xml
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="4000"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="360"/>
```

对动画效果的实现是通过属性动画来实现的，只是属性稍有不同。在 <group> 和 <path> 标签中添加 rotation、fillColor、pathData 等属性，在 objectAnimator 中可通过指定 android:propertyName="XXXX" 属性选择控制哪种属性，通过 android:valueFrom="XXX" 和 android:valueTo="XXX" 属性控制动画的起始值。如果指定属性为pathData，需要添加属性 android:valueType="pathType" 告诉系统进行 pathData 变换。类似可使用 rotation进行旋转动画，使用 fillColor 实现颜色动画，使用 pathData 进行形状、位置变化。

**（5）实例**

可将 propertyName 指定为 trimPathStart，该属性控制一个 SVG Path 的显示比例，例如一个圆形 SVG，使用
trimPathStart 动画，可像画出一个圆一样绘制一个圆，形成一个轨迹动画效果

### 八、Activity 与 Activity 调用栈分析

#### 1、Activity

任务栈（Task）中的 Activity 可以来自不同的 App，同一个 App 的 Activity 也可能不在一个 Task 中。

#### 2、Intent Flag 启动模式

系统提供两种方式设置 Activity 启动模式

> 通过设置 Intent 的 Flag 设置 Activity 启动模式，常用 Flag：

-  `Intent.FLAG_ACTIVITY_NEW_TASK` ：使用一个新 Task 启动一个 Activity，启动的每个 Activity 都在一个新 Task 中。该 Flag 通常使用在从 Service 中启动 Activity 的场景，由于 Service 中不存在 Activity 栈，所以使用该 Flag 创建一个新 Activity 栈，并创建新的 Activity 实例
-  `FLAG_ACTIVITY_SINGLE_TOP` ：使用 singletop 模式启动 Activity，与指定 `android:launchMode="singleTop"` 效果相同
-  `FLAG_ACTIVITY_CLEAR_TOP` ：使用 SingleTask 模式启动 Activity，与指定 `android:launchMode="singleTask"` 效果相同
-  `FLAG_ACTIVITY_NO_HISTORY` ：使用这种模式时当该 Activity 启动其他 Activity 后，该 Activity 也消失，不保留在 Activity 栈中

#### 3、清空任务栈

系统提供清空任务栈的方法将一个 Task 全部清除。可以在 AndroidMainifest 文件中的 <activity> 标签中使用以下几种属性来清理任务栈：

- clearTaskOnLaunch：在每次返回该 Activity 时，将该 Activity 之上所有 Activity 清除。该属性可让该 Task 每次初始化时，都只有这个 Activity
- finishOnTaskLaunch：与 clearTaskOnIaunch 属性类似，clearTaskOnIaunch 作用在别人身上，而finishOnTaskLaunch 作用在自己身上。当离开该 Activity 所处 Task，用户再返回时该 Activity 会被 finish
- alwaysRetainTaskState：若将 Activity 该属性设为 True，则该 Activity 所在 Task 将不接受任何清理命令，一直保持当前 Task 状态

### 九、Android 系统信息与安全机制

#### 1、Android 系统信息获取

> 获取系统配置信息，可以从以下两个方面获取：

- android.os. Build
- SystemProperty

**（1）android.os. Build**

> android.os. Build 类包含系统编译时设备、配置信息：

- Build.BOARD	                                   // 主板
- Build.BRAND                                       // Android系统定制商
- Build. SUPPORTED_ABIS                   // CPU指令集.
- Build.DEVICE                                       // 设备参数
- Build.DISPLAY                                     // 显示屏参数
- Build.FINGERPRINT                            // 唯一编号
- Build.SERIAL                                        // 硬件序列号
- Build.ID                                                // 修订版本列表
- Build.MANUFACTURER                     // 硬件制造商
- Build.MODEL                                      // 版本
- Build.HARDWARE                               // 硬件名
- Build.PRODUCT                                  // 手机产品名
- Build.TAGS                                          // 描述 Build 的标签
- Build.TYPE                                           // Builder类型
- Build.VERSION.CODENAME             // 当前开发代号
- Build.VERSION.INCREMENTAL        // 源码控制版本号
- Build.VERSION.RELEASE                  // 版本字符串
- Build.VERSION.SDK_INT                  // 版本号
- Build.HOST                                        // Host值
- Build.USER                                         // User名
- Build.TIME                                         // 编译时间

**（2）SystemProperty**

SystemProperty 包含许多系统配置属性值和参数，很多信息与通过 android.os.Build 获取的值相同

- os.version                         // OS版本
- os.name                            // OS名称
- os.arch                              // OS架构
- user.home                        // Home属性
- user.name                        // Name属性
- user.dir                              // Dir属性
- user.timezone                  // 时区
- path.separator                 // 路径分隔符
- line.separator                   // 行分隔符
- file.separator                    // 文件分隔符
- java.vendor.url                 // Java vender URL属性
- java.class.path                 // Java Class路径
- java.class.version            // Java Class版本
- java.vendor                      // Java Vender属性
- java.version                     // Java版本
- java.home                        // Java Home属性

**（3）Android 系统信息实例**

通过 android.os.Build 类，可直接获得一些 Build 提供的系统信息，通过 System.getProperty("XXXX") ，可访问到系统属性值

```java
String board = Build.BOARD;
String brand = Build.BRAND;
String os_version = System.getProperty("os.version");
String os_name = System.getProperty("os.name");
```

在 Android 系统目录的 system/build.prop 文件中，包含很多 RO 属性值，打开命令行窗口，进入 /system目录，通过 `cat build.prop` 命令查看文件信息，在 adb shell 中可通过 getprop 获取对应属性值

Android 系统有另外一个非常重要的目录来存储系统信息 —— /proc 目录，在 adb shell 进入 /proc 目录，通过 `ll` 命令查看文件信息，使用 `cat cpuinfo` 命令打开 cpuinfo 文件，这里的信息比通过 Build 获得的信息更加丰富，如果想获得更精确、丰富的系统信息，可通过执行 `adb shell` 命令查看这些节点文件获取更多系统信息

#### 2、Android Apk应用信息获取之PackageManager

在 ADB Shell 命令中，两个强大的命令集 —— PM 和 AM 命令，PM（PackageManager），AM（ActivityManager），PM 主宰应用包管理，AM 主宰应用活动管理

> PackageManager

<img src="Image.assets\package.jpg" alt="package" style="zoom:80%;" />

- 最里面的框代表整个 Activity 的信息，系统提供 ActivityInfo 类进行封装
- 最外面的框代表整个 Mainifest 文件中节点的信息，系统提供 PackageInfo 进行封装
- Android 系统提供 PackageManager 负责管理所有已安装的 App

> 常用封装信息

- ActivityInfo：封装了在 Mainifest 文件中 <activity></activity>  和 <receiver></receiver> 之间所有信息，包括 name、icon、label、launchmod 等
- ServiceInfo：与ActivityInfo类似，封装了 <service></service > 之间所有信息
- ApplicationInfo：封装 <application></application> 之间的信息，ApplicationInfo 包含很多 Flag，FLAG_SYSTEM 表示系统应用，FLAG_EXTERNAL_STORAGE 表示安装在 SDCard 上的应用等，通过这些Flag，可以方便地判断应用类型
- Packagelnfo：与前面三个 Info 类类似，用于封装 Mainifest 文件相关节点信息，PackagelInfo 包含所有Activity、Service等信息
- Resolvelnfo：封装包含 <intent> 信息的上一级信息，可以返回 ActivityInfo、ServiceInfo 等包含<intent> 的信息，经常用来找到那些包含特定 Intent 条件的信息，如带分享功能、播放功能的应用

> PackageManager 可以通过调用各种方法，返回不同类型的 Bean 对象。

- getPackageManager：返回一个 PackageManager 对象
- getApplicationInfo：以 ApplicationInfo 形式返回指定包名的 ApplicationInfo
- getApplicationlcon：返回指定包名的 lcon
- getlnstalledApplications：以 ApplicationInfo 形式返回安装的应用
- getInstalledPackages：以 PackageInfo 形式返回安装的应用
- queryIntentActivities：返回指定 intent 的 ResolveInfo 对象、Activity 集合
- queryIntentServices：返回指定 intent 的 ResolveInfo 对象、Service 集合
- resolveActivity：返回指定 intent 的 Activity
- resolveService：返回指定 intent 的 Service

判断 App 类型的依据是利用 ApplicationInfo 中的 FLAG_SYSTEM 进行判断 `app.flags & ApplicationInfo.FLAG_SYSTEM)` ，通过这样的标志区分，可判断出以下几种不同应用类型：

- flags & ApplicationInfo.FLAG SYSTEM != 0 则为系统应用
- flags & ApplicationInfo.FLAG SYSTEM <= 0 则为第三方应用
- 当系统应用经升级后，也将成为第三方应用：flags & ApplicationInfo.FLAG_UPDATED_SYSTEM_ APP != 0
- flags & ApplicationInfo.FLAG EXTERNAL_ STORAGE != 0 则为安装在 SDCard 上的应用

#### 3、Android Apk 应用信息获取之 ActivityManage

PackageManager 重点在获得应用包信息，ActivityManager 重点在获得在运行的应用程序信息。

ActivityManager 封装了不少 Bean 对象

> 第一个是内存信息

- ActivityManager.MemoryInfo：MemoryInfo 有几个重要字段：
  - availMem —— 系统可用内存
  - totalMem —— 总内存
  - threshold —— 低内存阈值（区分是否低内存的临界值）
  - lowMemory —— 是否处于低内存
- Debug.MemoryInfo：前面的 ActivityManager. MemoryInfo 通常用于获取全局的内存使用信息，Debug.MemoryInfo 用于统计进程下的内存信息
- RunningAppProcessInfo：运行进程信息，存储字段：
  - processName —— 进程名
  - pid —— 进程pid，uid —— 进程uid
  - pkgList —— 该进程下的所有包
- RunningServicelnfo：与 RunningAppProcessInfo 类似，用于封装运行的服务信息，其包含一些服务进程信息和其他信息。存储字段：
  - activeSince —— 第一次被激活的时间、方式
  - foreground —— 服务是否在后台执行

#### 4、解析 Packages.xml 获取系统信息

在系统初始化的时候，PackageManager 的底层实现类 PackageManagerService 会扫描系统中一些特定目录，解析其中的 Apk 文件，把获得的应用信息保存在 XML 文件中，做成一个应用花名册，系统中 Apk 安装、删除、升级时，其也会更新，其是位于 /data/system/ 目录下的 packages.xml 文件

> 信息点标签

- <permissions> 标签：定义目前系统中所有权限，分为两类：系统定义的（package 属性为 Android）和Apk 定义的（package 属性为 Apk 包名)
- <package> 标签：代表一个 Apk 的属性
  - name：Apk 包名
  - codcPath：Apk 安装路径，主要有 /system/app 和 /data/app 两种。/system/app 存放系统级别的 Apk或厂商定制 Apk，/data/app 存放用户安装的第三方 Apk
  - userId：用户ID
  - version：版本号
- <perms> 标签：对应 Apk 的 AndroidManifest 文件中的 <uses-permission> 标签，记录 Apk 权限信息

#### 5、Android 安全机制

**（1）Android安全机制简介**

> 五道防线

- 代码安全机制：代码混淆 proguard

  > 由于 Java 语言的特殊性，即使是编译成 Apk 的应用程序也存在被反编译的风险。proguard 可以混淆关键代码、替换命名让破坏者阅读困难，同时也可压缩代码、优化编译后的 Java 字节码

- 应用接入权限控制：AndroidMainifest 文件权限声明、权限检查机制

  > 任何 App 在使用 Android 受限资源时，都需显示向系统声明所需的权限，当 App 具有相应权限才能在申请受限资源的时候通过权限机制的检查并使用系统的 Binder 对象完成对系统服务的调用。

  > 这道防线有先天性不足:

  - 被授予的权限无法停止
  - 在应用声明 App 使用权限时，用户无法针对部分权限进行限
  - 权限的声明机制与用户的安全理念相关

  > Android 系统通常按照以下顺序检查操作者权限

  - 判断 permission 名称。为空直接返回 PERMISSION_DENIED
  - 判断 Uid。0 为 Root 权限，不做权限控制； System Server 的 Uid 为系统服务，不做权限控制；如果 Uid 与参数中的请求 Uid 不同，返回 PERMISSION_DENIED
  - 调用 `PackageManagerService.checkUidPermission()` 方法判断该 Uid 是否具有相应权限。该方法会去 XML 的权限列表和系统级的 platform.xml 中进行查找。

- 应用签名机制：数字证书

  > Android 中所有 App 都有一个数字证书（App 签名），数字证书用于保护 App 作者对其 App 的信任关系，只有拥有相同数字签名的 App，才会在升级时被认为是同一 App。且 Android 系统不安装没有签名的 App

- Linux 内核层安全机制：Uid、访问权限控制

  > Android 基于 Linux 内核开发，同样继承了 Linux 的安全特性，比如文件访问机制，Linux 文件系统的权限控制由user、group、other 与读(r)、写(w)、执行(x)的不同组合来实现。Android 也实现这套机制，通常情况下，只有 System、root 用户才有权限访问系统文件，一般用户无法访问

- Android 虚拟机沙箱机制：沙箱隔离

  > Android 的 App 运行在虚拟机中，因此才有沙箱机制，可以让应用之间相互隔离。通常情况下，不同应用之间不能互相访问，每个 App 有与之对应的 Uid ，每个 App 运行在单独的虚拟机中，与其他应用完全隔离。在实现安全机制的基础上，让应用之间互不影响，即使一个应用崩溃，也不导致其他应用异常。

**（2）Android 系统安全隐患**

i、代码漏洞

所有程序不敢保证不会有Bug、有漏洞，这种问题只能升级系统版本、更新补丁。比如 Android 的LaunchAnyWhere，FakelD 等 Bug，就是在代码编写的时产生的漏洞

ii、Root风险

Root 权限指 Android 的系统管理员权限，类似 Windows 系统的 Administrator。具有 Root 权限的用户可以访问和修改手机中几乎所有文件。Root 掉手机后可以解锁很多普通用户无法完成的工作，如限制各个 App 的数据流量、系统文件管理、自定义修改系统等，但同时手机的安全性也会因此大打折扣。随着 Android 系统越来越完善，普通用户在不 Root 的情况下，可以正常使用大部分 App。需要 Root 权限的大多为开发者由于开发需要。Root 后的手机少了一层 Linux 的天然屏障，整个系统核心完全暴露在入侵者面前

iii、安全机制不健全

Android 的权限管理机制并不完美，很多手机开发商通常会在 ROM 中增加自己的一套权限管理工具帮助用户控制手机中应用权限

vi、用户安全意识

v、Android 开发原则与安全

Android 与 iOS 系统显著的区别是一个是开放系统一个是封闭系统

开放的好处：技术进步快、产品丰富

封闭的好处：安全性高、可控性高

**（3）Android Apk 反编译**

Android 的 Apk 文件也是一个压缩文件，解压之后资源文件等 xml 文件基本都无法打开，打开也是乱码。这些乱码是经 Android 加密过的文件。解压后找不到源代码文件夹 src，只有 res 文件夹中查看非 XML 的图片资源文件，不过有些应用会把图片也加密处理。

i、apktool

> 利用 apktool_2.0.0b7.jar 反编译 Apk 中的 XML 文件

- 在命令行下进入其所在文件夹目录

- 对 XX.apk 执行反编译命令 `apktool_2.0.0b7.jar d XX.apk` 

  指定d参数（decode），写入要反编译的 Apk 目录。执行后在该目录下生成一个对应 Apk 名的文件夹。进入可查看相关反编译出来的代码

- 该工具在汉化软件的时候非常有用，可提取资源文件并进行汉化，然后重新打包回去即可 `apktoo1_2.0.0b7.jar b XX`

  重新打包的命令与解码的命令相似，只需将 d 改为 b，并选择解码生成的文件夹。执行后在文件夹下会生成两个新文件夹，重新打包的 Apk 在 dist 目录下，下一步解决 Source Code

ii、Dex2jar、jd-gui

解压 Apk 后生成的文件夹中有一个 dex 文件，该文件是源代码打包后的文件，将其复制到 dex2jar-0.0.9.15 的根目录下
反编译 dex 文件
 `dex2jar-0.0.9.15>d2j-dex2jar.bat classes.dex` 
分析 dex 文件
 `dex2jar classes.dex -> classes-dex2jar.jar` 

最后在 dex2jar-0.0.9.15 目录下生成了一个 jar 文件，打开 jd-gui，选择 file-open file，选择生成的 classes-dex2jar.jar 文件，就可查看相关源代码

**（4）Android Apk 加密**

Java 字节码的特殊性使它容易被反编译。为了对编译好的 Java Class 文件进行保护，通常会使用 ProGuard 对 Apk 进行混淆处理，用无意义字母重命名类、字段、方法、属性。ProGuard 不仅可用来混淆代码，还可删除无用类、字段、方法、属性、注释，最大限度优化字节码文件。在 Android Studio 中的 Gradle Scripts 文件夹下打开 build.gradle(Module: app) 文件，显示如下：

```groovy
buildTypes{
    release{
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'),'proguard-rules.pro'
    }
}
```

minifyEnabled 属性控制是否启用 ProGuard 的开关（以前叫 runProguard），在 AS1.1 中将其改为 minifyEnabled，设为 true 即打开 ProGuard 功能

proguardFiles 属性用于配置混淆文件，分为两部分，一是系统默认混淆文件，位于 <SDK 目录>/tools/proguard/proguard-android.txt 目录下，一般使用默认混淆文件即可，后面部分是项目中自定义的混淆文件，可在项目 App 文件夹下找到该文件，在该文件里可定义引入的第三方依赖包的混淆规则。配置好 ProGuard后，只要在使用 AS 导出 Apk 时即可生成混淆

### 十、Android 性能优化

#### 1、布局优化

**（1）Android UI 渲染机制**

人眼感觉的流畅画面需要画面帧数达到 40帧/s — 60帧/s，最佳 fps 大概在60fps左右。

Android 系统通过 VSYNC 信号触发对 UI 的渲染、重绘，间隔时间是16ms（1000ms中显示60帧画面的单位时间 —— 1000/60）。如果系统每
次渲染时间保持在16ms之内，需要将所有程序逻辑保证在16ms内。无法在16ms内完成绘制就会造成丢帧现象（当前该重绘的帧被未完成的逻辑阻塞），如一次绘制任务耗时20ms，在16ms系统发出 VSYNC 信号时无法绘制，该帧被丢弃，等待下次信号，导致 16*2ms 内显示同一帧画面，这是画面卡顿原因。

Android 系统提供检测 UI 渲染时间的工具，打开开发者选项，选择 Profile GPU Rendering，选中 On screen as bars 选项，屏幕上会显示条形图，每一条柱状线包含三部分，蓝色代表测量绘制 Display List 的时间，红色代表 OpenGL 渲染Display List 所需时间，黄色代表 CPU 等待 GPU 处理的时间。中间绿色横线代表
VSYNC 时间16ms，需尽量将所有条形图控制在这条绿线之下

**（2）避免 Overdraw**

Overdraw 过度绘制会浪费很多 CPU、GPU 资源，例如系统默认会绘制 Activity 背景，而如果再给布局绘制重叠的背景，那么默认 Activity 背景属于无效过度绘制 —— Overdraw。Android 系统在开发者选项中提供检测工具 Enablc GPU Overdraw。激活后可通过界面上颜色判断 Overdraw 的次数，应尽量增大蓝色区域、减少红色区域

**（3）优化布局层级**

Android 系统对 View 进行测量、布局、绘制是通过对 View 数的遍历操作的。如果 View 树高度太高会严重影响测量、布局和绘制的速度，优化布局第一个方法是降低 View 树高度，Google 建议 View 树高度不宜超过10层。

**（4）避免嵌套过多无用布局**

i、使用<include>标签重用Layout

很多界面都会存在一些共通 UI，比如 Topbar、Bottombar 等。如果每个界面都复制一段代码，不利于后期代码维护，增加程序冗余度。可以使用<include>标签定义一个共通 UI。

在<include>标签中可以使用 Layout 组件的一些属性来控制引用的布局。如果需在 <include> 标签中覆盖类似原布局中 android:layout_XXXXX 属性，必须在<include> 标签中同时指定android:layout_width 和 android:layout_height 属性

ii、使用 <ViewStub> 实现 View 的延迟加载

<ViewStub> 标签实现对一个 View 的引用并实现延迟加载。<ViewStub> 是一个轻量级组件，不仅不可视，且大小为 0。

与 <include> 标签类似，在主布局的 <ViewStub> 的 layout 属性引用目标布局，运行程序后 <ViewStub> 标签引用的布局没有显示，显示方法：先用 `findViewById()` 方法找到 <ViewStub> 组件，然后有两种方式显示该 View

- VISIBLE `mViewStub.setVisibility(View.VISIBLE);` 
- inflate `View inflateView = mViewStub.inflate();` 

两种方式唯一区别是 `inflate()` 方法可返回引用布局，从而可再通过 `View.findViewByld()` 方法找到对应控件

```java
View inflateView = mViewStub.inflate();
TextView textView = inflateView.findViewByld(R.id.tv);
```

一旦 <ViewStub> 被设为可见或被 inflate ，<ViewStub> 就不存在，取代的是被 inflate 的 Layout，将该Layout 的 ID 重新设为 <ViewStub> 通过 android:inflatedId 属性指定的 ID，所以两次调用 inflate 方法会报错

<ViewStub> 标签与 View.GONE 方式隐藏一个 View 的共同点是初始不显示，但 <ViewStub> 标签只在显示时才渲染整个布局，View.GONE 在初始化布局树时就已添加在布局树上，<ViewStub> 标签的布局具有更高的效率

**（5）Hierarchy Viewer**

通常情况下 Hierarchy Viewer 无法在真机使用，只能在工厂的 Demo 机和模拟器上使用（非加密过的设备），开源项目 View Server https://github.com/romainguy/ViewServer，通过该程序可让普通手机能使用 Hierarchy Viewer

在模拟器中使用该工具，其位于 sdk/tools 目录下，在命令行中输入 `hierarchyviewer.bat` 启动程序，选择要调试（测试）的进程，点击 Load View Hierarchy 按钮。通常情况下，关注 ID 为 content 的 FrameLayout 分支，这是 `setContentView()` 所设的内容，点击 View 时可显示该 View 绘制情况。第一次点击时候各种显示时间都是NA，需点击菜单中 Profile Node 按钮重新计算，才能获取绘制信息，此时可知每个 View 绘制时长，系统在下方给出了三个不同颜色的小圆点用来表示绘制效率，绿、黄、红分别代表好、中、差三种不同的绘制效率

#### 2、内存优化

**（1）内存**

由于 Android 应用的沙箱机制，每个应用分配的内存大小有限，内存太低会触发 LMK（Low Memory Killer）机制。通常说的内存指手机 RAM，其包括：

- 寄存器（Registers）：速度最快的存储场所，位于处理器内部，在程序中无法控制
- 栈（Stack）：存放基本类型的数据和对象的引用，当变量作用域结束后，这部分内存空间会马上被用作新的空间分配。对象本身不存放在栈中，而是存放在堆中
- 堆（Heap）：存放由 new 创建的对象和数组，堆中分配的内存由 Java 虚拟机的自动垃圾回收器（GC）管理，即使该对象作用域结束，这部分内存不会立即被回收，而是等待系统 GC 回收
- 静态存储区域（Static Field）：指在固定位置存放应用程序运行时一直存在的数据，Java 在内存中专门划分一个静态存储区域管理特殊的数据变量如静态数据变量。
- 常量池（Constant Pool）：JVM 虚拟机为每个被装载类型维护一个常量池。常量池是该类型用到常量的有序集合，包括直接常量（基本类型，String）和对其他类型、字段、方法的符号引用

> 获得堆大小，内存分析正是分析 Heap 中的内存状态

```java
ActivityManager manager = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
int heapSize = manager.getLargeMemoryClass();
```

**（2）获取Android系统内存信息**

i、Process Status

Process Stats 是 KK 一个系统内存监视服务，可通过 `Setting-Developer options-Process Stats` 开启该功能界面，也可使用 Dumpsys 命令获取信息 `adb shell dumpsys procstats` 

ii、Meminfo

Meminfo 是系统一个内存监视工具，可通过在 Settings—Apps—Running 中打开界面，也可使用 Dumpsys 命令 `adb shell dumpsys meminfo` 

**（3）内存回收**

Java 创建垃圾收集器线程（Garbage Collection Thread）来自动进行资源管理，Java 的 GC 是系统自动进行的，但何时进行是开发者无法控制的，调用 `System.gc()` 方法只是建议系统进行 GC。JVM虚拟机虽然能够自动控制GC，但也难免会存在部分对象忘记回收现象，这是造成内存泄漏的原因。

**（4）内存优化实例**

i、Bitmap 优化

Bitmap 是造成内存占用过高甚至 OOM（Out Of Memory）的最大威胁

- 使用适当分辨率和大小的图片

  Android 系统在做资源适配时会对不同分辨率文件夹下的图片缩放来适配相应分辨率，如果图片分辨率与资源文件夹分辨率不匹配或图片分辨率太高会导致系统消耗更多的内存资源。在图片列表界面可使用图片缩略图thumbnails，在显示详细图片的时再显示原图，在对图像要求不高的地方尽量降低图片精度

- 及时回收内存

  用完 Bitmap 后要及时使用`bitmap.recycle()` 方法释放内存资源。Android 3.0 后 Bitmap 被放置到堆中，内存由 GC 管理，不需释放

- 使用图片缓存

  通过内存缓存（LruCache）和硬盘缓存（DiskLruCache）可更好地使用Bitmap

ii、代码优化

任何 Java 类都占用大约 500 字节内存空间。创建一个类的实例会消耗大约 15 字节内存。小技巧：

- 对常量使用 static 修饰符
- 使用静态方法，静态方法比普通方法提高 15% 左右访问速度
- 减少不必要的成员变量，这在 Android Lint 工具上已集成检测，如果一个变量可定义为局部变量会建议你不定义为成员变量
- 减少不必要的对象，使用基础类型会比使用对象更节省资源，应避免频繁创建短作用域变量
- 尽量不用枚举、少用迭代器
- 对 Cursor、Receiver、Sensor、File 等对象，要注意其创建、回收、注册、解注册
- 避免使用 IOC 框架，IOC 通常使用注解、反射来进行实现，大量使用反射会带来性能的下降
- 使用 RenderScript、OpenGL 进行复杂绘图操作
- 使用 SurfaceView 替代 View 进行大量、频繁的绘图操作
- 尽量使用视图缓存，而不是每次都执行 `inflate()` 方法解析视图

#### 3、Lint工具

Android Lint 工具是 Android Studio 集成的一个 Android 代码提示工具

#### 4、使用 Android Studio 的 Memory Monitor 工具

Memory Monitor 工具是 Android Studio 自带的一个内存监视工具，可以进行内存实时分析。点击 Android Studio 右下角的 Memory Monitor 标签，打开 Memory Monitor 工具

较浅的蓝色代表 free 的内存，深色部分代表使用的内存从内存变换的走势图变换，可判断内存使用状态，当内存持续增高时可能发生内存泄漏，当内存突然减少时可能发生了 GC 等

#### 5、使用 TraceView 工具优化 App 性能

TraceView 是一个 Android 下的可视化性能调查工具，用来分析 TraceView 日志

**（1）生成 TraceView 日志的两种方法**

一是利用 Debug 类生成日志文件，另一个是利用 Android Device Monitor 工具辅助生成。

i、通过代码生成精确范围的 TraceView 日志

使用 Debug 类的方法开启 TraceView 监听。调用 `Debug.startMethodTracing()` 方法开启监听，调用 `Debug.stopMethodTracing()` 方法结束监听，使用这两个方法包围要监听的代码块，例如在 `onCreate()` 方法里调用 `startMethodTracing()` 方法开始监听，在 `onDestroy()` 方法方法里使用 `stopMethodTracing()` 方法结束监听。TraceView 日志保存到 /sdcard/ dmtrace.trace 目录下，需在 Mainifest 文件中增加权限：`<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE">` 。除了使用默认输出日志名，可自定义路径和日志名。当要监听内容执行完毕后通过 ADB 命令将日志文件导出到本地： `adb pull /sdcard/trace_log.trace/local/LOG/` 

ii、通过 Android Device Monitor 生成 TraceView 日志

打开 Android Device Monitor工具，选择要调试的进程，点击工具栏的 start method profiling 按钮，会弹出提示，选择监听模式，TraceView 提供两种监听方式：

- 整体监听：跟踪每个方法执行的全部过程，资源消耗较大
- 抽样监听：按照指定频率进行抽样调查，需要执行较长时间获取较准确的样本数据。执行一段时间后，再次点击 start method profiling 按钮即可结束监听

**（2）打开 TraceView 日志**

导出的 TraceView 日志文件，可使用 SDK 的 sdk\tools\traceview.bat 工具打开。或在 ADM 工具中，在 File 菜单下选择 Open File… 选项打开 TraceView 日志文件

**（3）分析 TraceView 日志**

TraceView 分析界面分为两部分，上面是显示方法执行时间的时间轴区域，下面是显示详细信息的profile区域

i、时间轴区域

时间轴区域显示不同线程在不同的时间段内的执行情况，在时间轴中每一行代表一个独立线程，不同色块代表不同的执行方法，色块长度代表方法执行时间

ii、profile区域

Profile 区域显示选择色块所代表的方法在该色块所处时间段内的性能分析

主要显示：

- Incl CPU Time：某方法占用CPU的时间
- Excl CPU Time：某方法本身（不包括子方法）占用 CPU 的时间
- InclReal Time：某方法真正执行的时间
- ExclReal Time：某方法本身（不包括子方法）真正执行的时间
- Calls + RecurCalls：调用次数 + 递归回调的次数

每个时间包含两列，一个是实际时间，一个是百分比。分析时通常从 Incl CPU Time 和 Calls + RecurCalls 开始，对占用时间长的方法进行重点分析，如果占用时间长且 Calls+RecurCalls 次数少，就可列为怀疑对象了

#### 6、使用 MAT（Memory Analyzer Tool）工具分析 App 内存状态

**（1）生成 HPROF 文件**

打开 Android Device Monitor 工具，选择要监听的线程，点击菜单栏的 Update Heap 按钮，在 Heap 标签中点击 Cause GC 按钮会显示当前内存状态

有个判断当前是否存在内存泄漏的小技巧：当不停点击 Cause GC 按钮时，如果 data object 一栏中的 Total Size 有明显变化就代表可能存在内存泄漏

上面是手动查看 Heap 状态，下面点击菜单栏的 Dump HPROF File 按钮，等待几秒钟后系统会生成一个 .hprof 文件，将它保存到 PC 上，默认名为包名.hprof。对该文件不能直接使用 MAT 工具分析，需要格式转换。在命令行下切换到 SDK 目录的 platform-tools 目录下，使用 hprof-conv 工具进行转换 `hprof-conv F:\Heap\com.imooc.heap.hprof heap.hprof` ，命令格式：hprof-conf infile outfile，生成的 heap.hprof  文件可以利用 MAT 工具进行内存分析

**（2）分析 HPROF 文件**

打开 MAT 工具，选择 Open a Heap Dump 选项，等待文件导入后，显示分析结果。

MAT 功能：

- Histogram（直方图）：用于显示内存中每个对象的数量、大小、名称。点击打开 Histogram 标签，在最上方一行可通过搜索过滤相应的关键字，在选择的对象上单击鼠标右键，在弹出的快捷菜单中选择 List objects-with incoming references 选项查看具体对象
- Dominator Tree（支配树）：将内存中对象按照大小排序，并显示对象之间的引用结构。点击打开Dominator Tree 标签，对象已按 Retained Heap 进行排序，即按对象及其所持有的引用的内存总和进行排序，分析内存占用大的对象找出内存消耗原因

#### 7、使用 Dumpsys 命令分析系统状态

使用 Dumpsys 命令可列出 Android 系统相关信息和服务状态。

使用 Dumpsys 命令：输入 adb shell dumpsys + 参数 。例如获取 Activity 栈详细信息 `adb shell dumpsys activity` 

> 常用 Dumpsys 参数

- activity：显示所有 Activity 栈信息
- meminfo：内存信息
- battery：电池信息
- package：包信息
- wifi：显示WiFi信息
- alarm：显示alarm信息
- procstats：显示内存状态

### 十一、搭建云端服务器

#### 1、移动后端服务

移动后端 —— Backend as a Service（Baas），Baas 把服务器端的东西全部打包，移动端不用考虑如何写服务器端、如何设计数据库、搭建服务器等，用户只需要调用API接口，就可以实现网络功能

#### 2、使用 Bmob 创建移动后端服务

Bmob 官网：http://www.bmob.cn/，下载 Bmob 的 Android SDK，将 BmobPush_V0.5beta_1027.jar 和BmobSDK_V3.2.6_1103.jar 拷贝到工程的 lib 目录下，按 F4 键选择 Module Setting，在 Dependencies 里添加刚加入的 lib。进入网站后台选择创建应用，创建后系统会生成三个唯一的 Key。

在 Mainifest 文件增加 Bmob 所需权限：

```xml
<uscs-permission android.name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>-
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_LOGS"/>
```


在应用程序的主入口中调用： `Bmob. initialize(this, "Application ID");` ，Application ID 是前面创建应用时生成的 Key

**（1）数据服务**

创建一个 BmobObject 对象。BmobObject 相当于数据库中的一张表，每个属性相当于表的字段，每一个BmobObject 对象相当于表里一行数据，通过面向对象方式操纵数据。

调用 BmobObject 的 save 方法可将 BmobObject 对象保存到云端服务器，同时产生两个回调方法获取 save 成功与失败的状态并进行相应操作

通过 BmobQuery 创建一个查询对象并使用 findObjects 方法获取数据，成功与失败两个回调方法可处理相应操作，通过 addWhereEqualTo 等方法相当于给 SQL 语句增加 Where 等条件语句

**（2）推送服务**

在 Mainifest 文件增加：

```xml
<receiver android:name="cn.bmob.push.PushReceiver"
	<intent-filter android:priority="2147483647”><!--优先级加最高-->
        <!--系统启动完成后会调用-->
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <!--解锁完成后会调用-->
        <action android:name="android.intent.action.USER_PRESENT"/>
        <!--监听网络连通性-->
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
	</intent-filter>
</receiver>

<!--自定义receiver，用来获取实现推送消息后的操作-->
<receiver android:name=".PushReceiver">
    <intent-filter>
    	<action android:name="cn.bmob.push.action.MESSAGE"/>
    </intent-filter>
</receiver>
```

在程序入口初始化 Push 服务：

```java
Bmoblnstallation.getCurrentInstallation(this).save();
BmobPush.startWork(this," Application ID ");
```

发送消息：

```java
BmobPushManager push = new BmobPushManager(MainActivity.this);
push.pushMessageAll("Test");
```

客户端获取信息：

```java
if (intent.getAction().equals(PushConstants.ACTION_MESSAGE)){
    String msg = intent.getStringExtra(PushConstants.EXTRA_PUSH_MESSAGE_STRING);
}
```

消息以 JSON 格式发送

### 十二、Android 5.X 新特性详解

#### 1、Android 5.X UI 设计

Android 5.X 系列开始使用 Material Design 统一 Android 系统界面设计风格。

- 材料的形态模拟：Google 通过模拟自然界纸墨形态变化、光线阴影、纸与纸之间的空间层级关系，带来空间感
- 更加真实的动画：Android 5.X 大量加入各种动画效果和各种新的转场动画
- 大色块使用：Material Design 用大量高饱和度、适中亮度的大色块突出界面主次，此外还有悬浮按钮、聚焦大图、无框按钮、波纹效果等新特性

#### 2、Material Design 主题

> 三种默认主题：

- @android:style/Theme.Material (dark version)
- @android:style/Theme.Material.Light (light version)
- @android:style/Theme.Material.Light.DarkActionBar

> 自定义：

```xml
<resources>
    <!-- inherit from the material theme -->
    <style name="AppTheme" parent="android:Theme.Material">
        <!-- Main theme colors -->
        <!-- your app branding color for the app bar -->
        <item name="android:colorPrimary">#BEBEBE</item>
        <!-- darker variant for the status bar and contextual app bars -->
        <item name="android:colorPrimaryDark">#FF5AEBFF</item>
        <!-- theme Ul controls like checkboxes and text fields -->
        <item name="android:navigationBarColor">#FFFF4130</item>
    </style>
</resources>
```

#### 3、Palette

Android 5.X 使用 Palette 来提取颜色，让主题能动态适应当前页面的色调，做到 App 颜色基调统一

> Android 内置几种提取色调种类：

- Vibrant（充满活力的）
- Vibrant dark（充满活力的黑）
- Vibrant light（充满活力的亮）
- Muted（柔和的）
- Muted dark（柔和的黑）
- Muted light（柔和的亮）

使用 Palette 添加依赖：`com.android.support:palette-v7:21.0.2` 

可传递一个 Bitmap 对象给 Palette，调用其 `Palette.generate()` 静态方法或 `Palette.generateAsync()` 方法创建一个 Palette。接下来可以使用 getter 方法检索相应色调

```java
Bitmap bitmap = BitmapFactory.decodeResource(getResources(),R.drawable.test);
//创建 Palette 对象
Palette.generateAsync(bitmap,new Palette.PaletteAsyncListener({
    @Override
    public void onGenerated(Palette palette){
        //通过Palette来获取对应的色调
        Palette.Swatch vibrant = palette.getDarkVibrantSwatch();
        //将颜色设置给相应的组件
        getActionBar().setBackgroundDrawable(new ColorDrawable(vibrant.getRgb());
        Window window = getWindow();
        window.setStatusBarColor(vibrant.getRgb());
    }
});
```

> 提取不同色调颜色

```java
palette.getVibrantSwatch();
palette.getDarkVibrantSwatch();
palette.getLightVibrantSwatch();
palette.getMutedSwatch();
palette.getDarkMutedSwatch();
palette.getLightMutedSwatch();
```

#### 4、视图和阴影

**（1）阴影效果**

Android 5.X 中，Google 在 X、Y 基础上增加了新属性 Z，对应垂直方向上的高度变化。View 的 Z 值由 elevation 和 translationZ 组成，elevation 是静态成员，translationZ 可在代码中使用实现动画效果

 `z = clevation + translationZ` 

在 XML 布局中静态设置 View 的视图高度： `android:clevation="XXdp"` 

在代码中动态改变视图高度： `view.setTranslationZ(XXX);` 

#### 5、Tinting（着色） 和 Clipping（裁剪）

**（1）Tinting（着色）**

在 XML 中配置 tint 和 tintMode 即可，Tint 通过修改图像 Alpha 遮罩修改图像颜色，实现重新着色

**（2）Clipping（裁剪）**

使用 Clipping 先用 ViewOutlineProvider 修改 outline，通过 setOutlineProvider 将 outline 作用给视图

```java
//获取Outline
ViewOutlineProvider viewOutlineProvider1 = new ViewOutlineProvider(){
    @Override
    public void getOutline(View view,Outline outline){
        //修改outline为特定形状
        outline.setRoundRect(0,0,view.getWidth(),view.getHeight(),30);
    }
};
```

#### 6、列表和卡片

**（1）RecyclerView**

**（2）CardView**

#### 7、Activity 过渡动画

>  Android 5.X 提供三种 Transition 类型

- 进入：一个进入的过渡动画决定 Activity 中所有视图怎么进入屏幕
- 退出：一个退出的过渡动画决定 Activity 中所有视图怎么退出屏幕
- 共享元素：一个共享元素过渡动画决定两个 Activities 之间的过渡，怎么共享它们的视图

> 进入和退出效果：

- explode（分解）：从屏幕中进 / 出，移动视图
- slidc（滑动）：从屏幕边缘进 / 出，移动视图
- fade（淡出）：通过改变屏幕上视图的不透明度达到添加或移除视图

> 普遍三种 Activity 过渡动画（A 跳转到 B）：

- A 中：`startActivity(intent,ActivityOptions.makeSceneTransitionAnimation(this).toBundle());` 

- B 中： `getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);`  或在样式文件中设置： `<item name="android:windowContentTransitions">true</item>`

- 设置进入 B 的具体动画效果：

  ```java
  getWindow().setEnterTransition(new Explode());
  getWindow().setEnterTransition(new Slide());
  getWindow().setEnterTransition(new Fade());
  ```

- 设置离开 B 的动画效果：

  ```java
  getWindow().setExitTransition(new Explode());
  getWindow().setExitTransition(new Slide());
  getWindow().setExitTransition(new Fade());
  ```

> 使用共享元素：

- 在 A 的布局文件中设置共享的元素，添加响应属性： `android:transitionName="XXX"` 

- 在 B 的布局文件中实现共享元素相同属性： `android:transitionName="XXX"` 

- 共享一个元素，在 A 中：

  ```java
  //参数是普通动画基础上增加共享的View和前面取的名字
  startActivity(intent,view,"share").toBundle());
  ```
  
- 共享元素

  ```java
  startActivity(intent,ActivityOptions.makeSceneTransitionAnimation(this,
                                                                    Pair.create(view, "share"),
                                                                    Pair.create(fab, "fab")).toBundle();
  ```

#### 8、Material Design 动画效果

**（1）Ripple 效果**

点击后波纹效果

可在布局文件加入：

```xml
//波纹有边界
android:background="?android:attr/selectableltemBackground"
//波纹超出边界
android:background="?android:attr/selectableltemBackgroundBorderless"
```

波纹有边界指波纹限制在控件边界中，波纹超出边界是波纹不限制在控件边界中，会呈圆形发散出去

可在 XML 文件中直创建一个具有 Ripple 效果的 XML 文件

```xml
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@android:color/holo_blue_bright">
    <item>
        <shape
        	android:shape="oval">
        	<solid android:color="?android:colorAccent"/>
        </shape>
    </item>
</ripple>
```

**（2）Circular Reveal**

该动画效果具体表现为一个 View 以圆形的形式展开、揭示出来。通过 `ViewAnimationUtils.createCircularReveal()` 方法可创建一个 RevealAnimator 动画

```java
public static Animator createCircularReveal(View view,
                                            int centerX,
                                            int centerY,
                                            float startRadius,
                                            float endRadius){
	return new RevealAnimator(view,centerX,centerY, startRadius,endRadius);
}
```

RevealAnimator 的使用主要是设置几个关键坐标点：

- centerX：动画开始中心点 X
- centerY：动画开始中心点 Y
- startRadius：动画开始半径
- endRadius：动画结束半径

**（3）View state changes Animation**

Android 5.X 系统提供视图状态改变来设置一个视图的状态切换动画

- StateListAnimator：作为视图改变时的动画效果，通常使用 Selector 进行设置，以前设置 Selector 通常修改背景达到反馈效果。Android 5.X 可使用动画作为视图改变效果

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <selector xmins:android="http://schemas.android.com/apk/res/android">
      <item android:state_pressed="true">
      	<set>
  		    <objectAnimator android:propertyName="rotationX"
                  android:duration="@android:integer/config_shortAnimTime"
                  android:valueTo="360"
                  android:valueType="floatType"/>
      	</set>
      </item>
      <item android:state_pressed="false">
      	<set>
              <objectAnimator android:propertyName="rotationX"
                  android:duration="@android:integer/config_shortAnimTime"
                  android:valueTo="0"
                  android:valueType="floatType">
      	</set>
      </item>
  </selector>
  ```

  在代码中可调用 `AnimationInflater.loadStatcListAnimator()` 方法，通过 `View.setStateListAnimator()` 方法分配动画到视图上

- animated-sclector：是一个状态改变的动画效果 Selector。Android 5.0 很多 Material Design 的控件设计是通过该方式实现，例如 check_box 的动画效果是使用类似帧动画的切换效果模拟进行点击时的切换效果，与 selector 的使用类似是通过 <item> 标签区分不同状态

#### 9、Toolbar

Toolbar 与 Actionbar 的区别是 Toolbar 更自由、可控。使用 Toolbar 须引入 appcompat-v7 支持，设置主题为
NoActionBar

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- toolbar颜色 -->
        <item name="colorPrimary">#4876FF</item>
        <!-- 状态栏颜色 -->
        <item name="colorPrimaryDark">#3A5FCD</item>
        <!-- 窗口的背景颜色 -->
        <item name="android:windowBackground">@android:color/white</item>
        <!-- add SearchView -->
        <item name="searchViewStyle">@style/MySearchView</item>
    </style>
    
    <style name="MySearchView" parent="Widget.AppCompat.SearchView"/>
</resources>
```

> 添加 Toolbar 显示的标题和图标

```java
mToolbar = findViewByld(R.id.toolbar);
mToolbar.setLogo(R.drawable.ic_launcher);
mToolbar.setTitle("主标题");
mToolbar.setSubtitle("副标题");
setSupportActionBar(mToolbar);//模拟出Actionbar效果
```

#### 10、Notification

Notification 作为一个事件触发通知型的交互提示接口，获得消息时在状态栏、锁屏界面得到相应提示信息。Google 在 Android 5.0 进一步改进通知栏，优化 Notification。当长按 Notification 时会显示消息来源，Notification 有一个从白色到灰色的动画切换效果，显示发出该 Notification 的调用者。Android 5.X 的设备锁屏状态下也可见 Notification 通知

**（1）折叠式 Notification**

折叠式 Notification 是一种自定义视图 Notification，常用于显示长文本。拥有两个视图状态，一是普通状态视图状态，另一个是展开状态视图状态。使用 RemoteViews 创建自定义 Notification 视图

```java
//通过 RemoteViews来创建自定义的Notification视图
RemoteViews contentView = new RemoteViews(getPackageName(),R.layout.notification);
contentView.setTextViewText(R.id.textView, "show me when collapsed");
```

指定正常状态下的视图： `notification.contentView = contentView;` 

指定展开时的视图： `notification.bigContentView = expandedView;` 

**（2）悬挂式 Notification**

Notification 可在屏幕上方产生 Notification ，不打断用户操作

```java
Notification.Builder builder = new Notification.Builder(this)
                            .setSmalllcon(R.drawable.ic_launcher)
                            .setPriority(Notification.PRIORITY_DEFAULT)
                            .setCategory(Notification.CATEGORY_MESSAGE)
                            .setContentTitle("Headsup Notification")
                            .setContentText("I am a Headsup notification.");
Intent push = new Intent();
push.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
push.setClass(this,MainActivity.class);
PendingIntent pendinglntent = Pendinglntent.getActivity(this,0,push,PendingIntent.FLAG_CANCEL_CURRENT);
builder.setContentText("Heads-Up Notification on Android 5.0").setFullScreenIntent(pendingIntent,true);
NotificationManager nm = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
nm.notify(NOTIFICATION_ID_HEADSUP, builder.build());
```

**（3）显示等级的Notification**

- VISIBILITY_PRIVATE：只有没锁屏时显示
- VISIBILITY_PUBLIC：任何情况都显示
- VISIBILITY_SECRET：在 pin、password 等安全锁和没锁屏的情况下才能显示

> 设置： `builder.setVisibility(Notification.VISIBILITY_PUBLIC);` 

> Notification 背景颜色接口： `builder.setColor(Color.RED)` 

> Notification 的 category 接口（category 确定其显示位置，参数是 category 类型）： `builder.setCategory(Notification.CATEGORY_MESSAGE)` 
