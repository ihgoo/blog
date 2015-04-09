title: 初识事件订阅之otto
date: 2015-03-04 11:04:15
categories: Android
---
###介绍
**Otto**是square的一个Event Bus模式的开源类库，事件订阅模式旨在当应用功能越来越多的时候，保证应用的各个部分之间高效通信变的简洁。通过事件订阅模式，应用组件间可以大大减少耦合性。
<!-- more -->
###配置
Otto项目地址在[这里](http://square.github.io/otto/)
在这里我以Gradle方式导入

	dependencies {
	  compile 'com.squareup:otto:+'
	}

用eclipse也可以直接用jar包，地址在[这里](https://search.maven.org/remote_content?g=com.squareup&a=otto&v=LATEST)
###使用
使用场景，我们需要在某界面中接收各个事件，以便更新UI，或者是两个组件之间进行通讯。类似于内容观察者，但又不等同于它，比内容观察者使用简单，耦合性少。

在这里注册事件Event，告诉它，现在要接收各个事件的更新了。而不需要关注在哪发送的事件更新的通知。

	     bus = new Bus();
	     bus.register(this);


用@Suscribe注解标记这个方法为可被事件订阅调用的方法，如果发送关于CommentEvent的事件，EventBus就会把所有关于CommentEven的事件发给这个方法。


	    @Subscribe
	    public void toggleKeyboard(CommentEven event) {

	        LogUtils.e(event.getCommentSomeBody());
	        mEtSerach.setHint(event.getCommentSomeBody());
	        mEtSerach.requestFocus();
	        mEtSerach.setFocusableInTouchMode(true);
	        KeyboardUtil.toggle(this, mEtSerach);
	        if (event.getType().equals("1")){
	            type = 1;
	        }else if(event.getType().equals("2")){
	            type = 2;
	        }
	    }


发送关于CommentEven的事件


	    @OnClick(R.id.ll_comment_comment)
	    void comment(){
	        CommentEven commentEven = new CommentEven("发表你的评论","2");
	        bus.post(commentEven);
	    }

取消订阅


	    @Override
	    public void onDestroy(){
	        super.onDestroy();
	        bus.unregister(this);
	    }

在这儿可以用单例模式创建Eventbus,[引用代码](https://github.com/bkiers/otto-demo/blob/master/src/main/java/com/askcs/ottodemo/service/ForegroundService.java)
	
	
	/**
	 * Maintains a singleton instance for obtaining the event bus over
	 * which messages are passed from UI components (such as Activities
	 * and Fragments) to Services, and back.
	 */
	public final class BusProvider {

	    // The singleton of the Bus instance which can be used from
	    // any thread in the app.
	    private static final Bus BUS = new Bus(){

	        private final Handler mainThread = new Handler(Looper.getMainLooper());

	        @Override
	        public void post(final Object event) {

	            if (Looper.myLooper() == Looper.getMainLooper()) {
	                super.post(event);
	            }
	            else {
	                mainThread.post(new Runnable() {
	                    @Override
	                    public void run() {
	                        post(event);
	                    }
	                });
	            }
	        }
	    };

	    /**
	     * Returns a singleton instance for obtaining the event bus over
	     * which messages are passed from UI components to Services, and
	     * back.
	     *
	     * @return a singleton instance for obtaining the event bus over
	     * which messages are passed from UI components to Services, and
	     * back.
	     */
	    public static Bus getBus() {
	        return BUS;
	    }

	    // No need to instantiate this class.
	    private BusProvider() {
	    }
	}