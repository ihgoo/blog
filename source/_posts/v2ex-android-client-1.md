title: "V2EX论坛客户端之帖子信息爬取(一)"
date: 2015-05-24 11:47:28
categories: Android
---
##前言
由于逛V2EX比较多，决定用闲暇时间做一个安卓客户端,开源在这里 https://github.com/ihgoo/V2EX

工欲善其事，必先利其器，整个项目以Gradle方式构建，Androidstudio开发。
公司的项目也转向AndroidStudio了，studio有个我特别喜欢的特性，输入变量的时候 记不住变量开头是怎么拼写的，能记住后面也会自动提示出来！还有就是插件多，开发向傻瓜化发展，只关注业务逻辑即可。
 <!--more-->
##按照业务分模块
论坛客户端按照业务逻辑会分为以下模块

- 在非登录状态下的浏览模块
	- 帖子列表浏览
	- 帖子详情浏览

- 用户模块
	- 登录模块
	- 用户信息模块

- 在登录状态下的模块
	- 带登录状态的回帖，帖子详情浏览
	- 收藏，点赞
	- 回帖提醒

##依赖
依赖库会尽量考虑使用原生控件以及成熟的框架

    compile 'com.jakewharton:butterknife:6.1.0'    
    compile 'com.squareup.retrofit:retrofit:1.9.0'
    compile 'com.squareup:otto:+'
    compile 'com.facebook.fresco:fresco:0.1.0+'
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.0.0'
    compile 'com.squareup.okhttp:okhttp:2.0.0'

butterknife：jack大神写的Ioc框架，媲美dagger，在idea/studio上面有支持butterknife的插件，一键findviewbyid！
![ezelezny_animated](https://github.com/avast/android-butterknife-zelezny/raw/master/img/zelezny_animated.gif)

retrofit: 强大的网络请求库。

fresco: 加载图片库，在使用这个之前，都是使用Imageloader的，刚出没多久的图片库，使用它是因为在项目中会有支持gif和支持图片渐进式显示的需求。

otto：eventBus框架！解耦神器，有了它，一切都变得简单了起来。

##搭建项目，整理包结构
包结构如下图：

![](http://ihgoo.qiniudn.com/CB99A173-56B4-4B7E-9375-CAAF49288689.png)

app：关于app的application等。
client：网络请求的报文头的定义，网络请求库的配置等。
core：基础框架，相当于mvc结构中的c，当然这里的c是指Controller
model：模型层
paser：解析层。无论是json，还是html，都是由此层解析生成实体类的。
persistence：放了一些常量类，数据库字段，Intnet请求字段，app配置字段等。
ui：视图展示层。
utils：一些顺手的工具类

项目以Gradle构建，app作为一个module，其他module作为挂载的形式挂到app上，优点是其他module可快速替换，且源码可修改（aar形式导入源码不可修改）。

##关于API
由于调用官方json api有调用次数限制，于是考虑采用解析html页面来做。
电脑端html太过于庞大，为了省电降低app占用资源，解析的是wap端的页面，
可以通过修改请求头里的UA字段伪装成手机浏览器，在这里我用的是

	public class ApiHeaders implements RequestInterceptor {
	    private String sessionId;

	    public void setSessionId(String sessionId) {
	        this.sessionId = sessionId;
	    }

	    public void clearSessionId() {
	        sessionId = null;
	    }

	    @Override public void intercept(RequestFacade request) {
	        request.addHeader("User-Agent", "Mozilla/5.0 (Linux; Android 4.1.1; Nexus 7 Build/JRO03D) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.166  Safari/535.19");
	        request.addHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8");
	        if (sessionId != null) {
	        }
	    }
	}


解析html，java里可以用一个叫做jsoup的库，媲美python中的pyquery，支持类Jqery选择器方式来抓取自己想要的资源，简单方便粗暴极了。
唯一的缺点就是如果页面里有些元素是ajax形式形成的，那这个就抓瞎了，可以使用httpunit，不过httpunit存在性能问题，要启动一个浏览器内核来运行这个网页，网页上的js完成后，再抓取信息。

##解析帖子列表
在帖子列表中需要关注解析这几个数据
- avatar 作者头像
- node	节点名称
- title 标题
- small fade (time) 发表时间
- small fade author 作者名称
- count_livid 回帖数
以下是用jsoup解析的代码，解析成功后塞到ForumItemBean这个实体类中，以集合形式返回给listView的adapter中

		public class PaserFourmList {

		    public static ArrayList<ForumItemBean> paser2ForumItem(String string) {
		        Document document = Jsoup.parse(string);
		        Elements elements = document.select(".cell").select(".item");
		        ArrayList<ForumItemBean> list = new ArrayList<>();
		        for (Element element : elements) {
		            // avatar
		            // node
		            // title
		            // small fade (time)
		            // small fade author
		            // count_livid
		            String avatar = element.select(".avatar").first().attr("src");
		            String node = element.select(".node").first().html();
		            String username = element.select(".small > strong").first().text();
		            String countLivid = element.getElementsByClass("count_livid").text();
		            String time = element.select(".small").select(".fade").get(1).text();

		            String href = element.getElementsByClass("item_title").html();
		            if (href.length()!=0){
		                href = href.substring(12, href.indexOf("#"));
		            }

		            int indexOf = time.indexOf("前");
		            if (indexOf != -1) {
		                time = time.substring(0, indexOf);
		            }

		            ForumItemBean forumItemBean = new ForumItemBean();
		            Member member = new Member();
		            member.setAvatarMini(avatar);
		            member.setUsername(username);
		            forumItemBean.setId(Misc.parseInt(href, 0));
		            forumItemBean.setMember(member);
		            forumItemBean.setLastTime(time);
		            forumItemBean.setReplies(Misc.parseInt(countLivid, 0));
		            forumItemBean.setTitle(element.select(".item_title").first().select("[href]").html());
		            list.add(forumItemBean);
		        }
		        return list;

		    }

		}


##使用Jsoup遇到的坑
在用jsoup的时候 //<//div class="content type"><///div>像这种class带空格的，需要使用 element.select(".content").select(".type")，才可以成功解析，使用element.select(".content type")是解析不出来的！

还有  //<//div class="content-type"><///div>   ，这种的，使用element.select(".content-type")也解析不出来，需要用element.getElementsByClass("content-type")才可以。










