---
layout: post
title: Android开发者必知的5个开源库
category: android
description: Gson, Retrofit,EventBus,ActiveAndroid,UNIVERSAL IMAGE LOADER
---

过去的时间里，Android开发逐步走向成熟，一个个与Android相关的开发工具也层出不穷。不过，在面对各种新鲜事物时，不要忘了那些我们每天使用的大量开源库。在这里，向大家介绍的就是，在这个任劳任怨的大家庭中，最受开发者喜爱的五个Android库。希望通过对它们的了解，能够对你的开发工作有所帮助。

## GSON

Gson是Google提供的用来在Java对象和JSON数据之间进行映射的Java类库。可用于将Java对象转换成对应的JSON表示，也可以将JSON字符串转换成一个等效的Java对象。如果与API打交道的话，那么这将会是你经常需要的东西。我们主要使用JSON的原因就是，相较XML，轻量级的JSON要简单的多。


	String userJSON = new Gson().toJson(user); 
 	  	  
	User user = new Gson().fromJson(userJSON, User.class); 



##RETROFIT

就如它网站上的介绍“Retrofit将你的REST API变为Java接口”一样，Retrofit把REST API返回的数据转化为Java对象方便操作，对于在项目中组织API调用，是一个不错的解决方案。其请求方法和相对URL都带有注解，使得代码变得更加简洁。使用注解，你可以很容易的添加一个请求主体，操纵URL或头文件，并添加查询参数。除此之外，每个函数可以定义为同步或异步，具有返回值的函数为同步执行，而异步函数没有返回值且最后一个参数为Callback对象。

Retrofit默认情况下使用的是GSON，所以无需自定义解析，同时还支持其他转换器。

## EVENTBUS

EventBus是用于简化应用中各个部件之间通信的一个库。比如从一个Activity发送消息到一个正在运行的服务，亦或是片段之间简单的互动。而下面使用的示例，就是如果网络连接丢失，该如何通知一个活动：

	public class NetworkStateReceiver extends BroadcastReceiver {
	
	    // post event if there is no Internet connection
	    public void onReceive(Context context, Intent intent) {
	        super.onReceive(context, intent);
	        if(intent.getExtras()!=null) {
	            NetworkInfo ni=(NetworkInfo) intent.getExtras().get(ConnectivityManager.EXTRA_NETWORK_INFO);
	            if(ni!=null && ni.getState()==NetworkInfo.State.CONNECTED) {
	                // there is Internet connection
	            } else if(intent
	                .getBooleanExtra(ConnectivityManager.EXTRA_NO_CONNECTIVITY,Boolean.FALSE)) {
	                // no Internet connection, send network state changed
	                EventBus.getDefault().post(new NetworkStateChanged(false));
	            }
	}
	
	// event
	public class NetworkStateChanged {
	
	    private mIsInternetConnected;
	
	    public NetworkStateChanged(boolean isInternetConnected) {
	        this.mIsInternetConnected = isInternetConnected;
	    }
	
	    public boolean isInternetConnected() {
	        return this.mIsInternetConnected;
	    }
	}
	
	public class HomeActivity extends Activity {
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        EventBus.getDefault().register(this); // register EventBus
	    }
	
	    @Override
	    protected void onDestroy() {
	        super.onDestroy();
	        EventBus.getDefault().unregister(this); // unregister EventBus
	    }
	
	    // method that will be called when someone posts an event NetworkStateChanged
	    public void onEventMainThread(NetworkStateChanged event) {
	        if (!event.isInternetConnected()) {
	            Toast.makeText(this, "No Internet connection!", Toast.LENGTH_SHORT).show();
	        }
	    }
	
	}



##ACTIVEANDROID
ActiveAndroid算是一个轻量级的ORM（对象关系映射），让你无需编写一个单独的SQL语句，就可以保存和检索SQLite数据库记录。每个数据库记录都被包裹整齐地归为一类，如delete（）和save（）的方法。

## UNIVERSAL IMAGE LOADER

UIL是是一个开源项目，其目的就是提供一个可重复使用的仪器为异步图像加载、缓存和显示。它的使用很简单：

	imageLoader.displayImage(imageUri, imageView);  

尽管Picasso拥有更好的API，但其缺乏自定义。而使用UIL构建器几乎可以配置所有（其中最重要的就是在抓取和缓存大型图片时，Picasso会失败）。

良好的开源库会让你的开发变得更简单更快速，而普遍流行的库通常测试良好且易用使用。在大多情况下，你可以很容易的将它们从Maven中导入到Android Studio项目里。将它们添加到相关性的build.gradle 文件。并且同步之后，在你的应用里将能够很好的实现它们。