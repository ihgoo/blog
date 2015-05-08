title: Android 开发规范及性能优化(一)
date: 2015-03-05 09:40:48
categories: Android
---
##**编程原则**
----

1.尽可能用自己所知道的最简单的方法解决问题。
2.做好注释。在复杂的类或者方法中，做好用途、用法注释；算法内部做好重要流程注释等。
<!-- more -->
3.处理Warning级别以上的提供，保持代码的干净整洁。
4.随时清理自己的测试逻辑与Log输出
5.有疑问或需要注意、重构的地方增加 //TODO 标识，同时做好相关注释
6.需要重构的类或方法，需做好备份，确保无误后，再进行删除
7.明确需求。开发前需与相关人员明确需求，整理并确认方案后，方可着手Coding
8.注意沟通。经常与相关人员进行沟通，确保自己工作的准确实时，以及与其他平台软件的统一。

##**开发规范**
----

1.需要在界面中显示的文字，需注意做好多语言

2.局部变量命名，静态成员变量命名
只能包含字母，单词首字母 除第一个外，都为大写，其他字母都为小写 

3.常量命名
只能包含字母和_，字母全部大写，单词之间用_隔开

4.layout中的id命名
命名模式为：view 缩写_模块名称_view的逻辑名称
view 的缩写详情如下
LayoutView :lv
RelativeView: rv
TextView: tv
ImageView: iv
ImageButton: im
Button: btn

5.activity中view的变量命名
命名模式为view缩写+ 逻辑名称(首字母大写)

6.strings.xml的id命名
命名模式：activity名称_功能模块名称_逻辑名称/activty名称_逻辑名称
strings.xml中，使用activity名称注释，将文件内容区别开来

7.drawable中的图片命名
命名模式： activity名称_逻辑名称/common_逻辑名称

8.styles.xml
将layout中不断重现的style提炼出通用的style通用组件，放到styles.xml中;

9.使用layer-list和selector

10.图片尽量分拆成多个可重用的图片

11.服务器可以实现的，就不要放在客户端(可以与后端协商实现)

12.引用第三方库要慎重，避免应用大容量的第三方库，导致客户端包非常大

13.图片的.9处理，注意图片的可扩展性，与适配效果

14.使用静态变量方法实现界面间共享要慎重

15.打印Log需使用Misc中的方法，防止正式版出现测试log

16.单元测试(逻辑测试、界面测试)

17.不要重用父类的handler，对应一个类的handler也不应该让其子类用到，否则会导致message.what冲突

18.activity中的一个View.OnclickListener中处理所有的逻辑

19.strings.xml中使用%1$s实现字符串的通配

20.如果多个Activity中包含共同的UI处理，那么可以提炼一个CommonActivity，把通用部分叫由它来处理，其他Activity只要继承它即可

21.使用button+activitygroup实现tab效果时，使用Button.setSelected(true)，确保按钮处于选择状态，并使activitygroup的当前activity与该button对应

22.如果所开发的为通用组件，为避免冲突，将drawable/layout/menu/values目录下的文件名增加前缀

23.数据一定要校验，例如
字符型转数字型，如果转换失败一定要有缺省值；
服务端响应数据是否有效判断；

24.同一个客户端如果要放在不同的市场，而且要统计各个市场下载及使用数据时，针对不同的客户端打不同的包

25.有的按钮要避免重复点击

26.数据库及文件操作，注意关闭通道

27.与服务器的数据传输格式为Json，通用字段检测：err,timestamp，errdesc

##性能优化
1.网络通讯需做好异步操作