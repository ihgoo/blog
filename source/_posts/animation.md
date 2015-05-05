title: "Android动画之属性动画"
date: 2015-05-04 13:34:27
categories: Android
---
###动画概述
Android为动画效果提供了一个强大多样化的Api，你可以根据你需要的来选择。


##Animation
Android框架层提供了两种动画系统：View Animation和Property Animation(Android3.0引入),除了属性动画以外动画系统都需要提前配置，通常使用的时候先要声明参数及方法，因为它很灵活，且提供了很多特性。除了以上两种动画系统之外，你还可以使用Drawable Animation，通过加载一连串图片来实现一些复杂动画。
<!--more-->

- PropertyAnimation
    在Android 3.0(API level11),这套系统可以使任意对象的任意属性进行动画变换，即便这个对象不在屏幕上面，这套系统可以让你的自定义动画达到极致。

- ViewAnimation
	视图动画是一套旧的系统且只能被应用在View上面的动画。它相对容易去实现，能满足大部分app的需要。

- DrawableAnimation
	帧动画像胶片电影一样，用一个接一个的显示drawable资源来达到动画的效果，这种动画实现起来很简单，缺点是播放这种动画消耗系统资源较多。

##Property Animation

属性动画系统可以让你为动画定义以下属性
- Duration
为动画设置持续时间，默认为300毫秒

- Time interpolation
时间插值，可以定义随着时间的改变，属性也进行改变。

- Repeat count and behavior
重复的次数

- Animator sets
动画集合，可以设置顺序执行动画，也可以同时执行动画

- Frame refresh delay
帧刷新的时间，默认为10ms/帧

##Property Animation是如何工作的
首先，用一个简单的例子来说明动画是如何工作的：

- 线性动画

![](http://ihgoo.qiniudn.com/animation-linear.png)

一个物体以1ms 1单位长度的速度前进，共前进40ms。

- 非线性动画

![](http://ihgoo.qiniudn.com/animation-nonlinear.png)

一个物体先加速后减速，共前进40ms。

- 动画是如何计算的

以上两种动画用属性动画都可以用属性动画来实现，每个物体拥有自己的速度属性，根据速度属性不同，前进的距离、速率也不同，实现的动画自然也就不同了，下面看具体的详解图：

![](http://ihgoo.qiniudn.com/valueanimator.png)

- ValueAnimation对象持续跟踪animation的时间点，如animation运行了多长时间，属性的当前值。
-  ValueAnimator封装了一个TimeInterpolator，TimeInterpolator定义了animation是如何插值的。
ValueAnimator也封装了一个TypeEvaluator，TypeEvaluator定义了如何计算属性的值。
在例子2中，TimeInterpolator使用的是先快后慢的插值器，TypeEvaluator使用的是IntEvaluator。
- 创建了一个ValueAnimator并为ValueAnimator注入animation的开始值，结束值，持续时间，调用start();方法来启动这个动画。
- 在Animation的整个运行过程中，ValueAnimator会基于动画运行的时间计算出一个0-1的值，这个值被称为elapsed fraction。如场景一t=10ms的时刻，elapsed fraction的值为0.25，因为整个动画的运行时间是40ms，所以elapsed fraction=10/40。
- 在ValueAnimator完成elapsed fraction的计算之后，它会调用TimeInterpolator去计算interpolated fraction的值。interpolated fraction会根据elapsed fraction的值来计算。例如，在场景二中，由于是缓慢加速的原因，在t=10ms的时刻，interpolated fraction的值为0.15，会小于elapsed fraction的值0.25。而在场景一中，由于是匀速的关系，interpolated fraction的值会永远与elapsed fraction的值一样的。
- 在ValueAnimator完成elapsed fraction的计算之后，它会调用合适的TypeEvaluator去计算属性的当前时刻的值，这个计算过程是基于interpolated fraction的值、starting value和ending value。例如，在场景二t=10ms的时刻点，interpolated fraction的值为0.15，因此属性x的值便会是0.15X(40-0)=6.


##ValueAnimator
ValueAnimator是属性动画的核心类，它可以设置动画持久时间，开始结束的属性值，相应时间属性值计算方法。

	ValueAnimator anim= ValueAnimator.ofInt(0, 40);  
	animation.setDuration(40);  
	animation.start(); 

然后通过ValueAnimator的AnimatorUpdateListener可以得到动画执行每一帧所返回的值,

	anim.addUpdateListener(new AnimatorUpdateListener() {  
	    @Override  
	        public void onAnimationUpdate(ValueAnimator animation) {  
	                //frameValue是通过startValue和endValue以及Fraction计算出来的  
	       	 int frameValue = (Integer)animation.getAnimatedValue();  
	        //利用每一帧返回的值，可以做一些改变View大小，坐标，透明度等等操作  
	    }  
	});  


##ObjectAnimator
ObjectAnimator
而ObjectAnimator是继承于ValueAnimator的，ObjectAnimator与ValueAnimator的不同区别在于不需要自己实现AnimatorUpdateLinstener接口来改变物体的状态的移动。

ObjectAnimator可以设置一个物体的动画属性值，然后提供该值的get/set方法，在动画一开始的时候，该动画会通过反射的方法得到/设置该值。

##AnimatiorSet
AnimatiorSet顾名思义就是几个动画的集合，这些动画可以一起执行，也可以顺序执行。
执行playTogether(Animator... items)或playTogether(Collection<Animator> items)方法来一起执行这些动画。
执行playSequentially(Animator... items)或playSequentially(List<Animator> items)方法来顺序执行这些动画。


##Interpolator
Interpolator作为动画的插值器，动画是如何变化的，都是通过插值器来决定的。

- AccelerateDecelerateInterpolator 先加速后减速
- AccelerateInterpolator 加速
- AnticipateInterpolator 
- AnticipateOvershootInterpolator 回荡秋千插值器
- BounceInterpolator 弹跳插值器
- CycleInterpolator 速率根据正弦曲线改变
- DecelerateInterpolator 减速
- LinearInterpolator 线性速度(匀速)
- OvershootInterpolator 

系统提供的插值器可以方便使用，不需要自己写数学函数了，如果需求不仅仅于上面几种插值器，也可以自定义插值器来实现


关于插值器的更多资料阅读  

- [简单插值器分析](http://my.oschina.net/banxi/blog/135633#OSC_h2_7)
- [动画初步使用以及自定义插值器](http://www.cnblogs.com/boliu/archive/2013/09/02/3295944.html)