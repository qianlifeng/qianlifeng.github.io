title: Android之旅---广播（BroadCast）
date: 2011-03-06 18:15:17
tags: [android]
---

本文介绍了如何使用pyinstaller打包python程序，使其成为window下的单个可执行exe文件。

<!-- more -->

#什么是广播
---------------
在Android中，Broadcast是一种广泛运用的在应用程序之间传输信息的机制。我们拿广播电台来做个比方。我们平常使用收音机收音是这样的：许许多多不同的广播电台通过特定的频率来发送他们的内容，而我们用户只需要将频率调成和广播电台的一样就可以收听他们的内容了。Android中的广播机制就和这个差不多的道理。

电台发送的内容是语音，而在Android中我们要发送的广播内容是一个Intent。这个Intent中可以携带我们要传送的数据。

电台通过大功率的发射器发送内容，而在Android中则是通过sendBroadcast这个方法来发送（很形象的名字吧）。       

用户通过调整到具体的电台频率接受电台的内容。而在Android中要接受广播中的内容则是通过注册一个BroadCastReceiver来接收的。只有发送广播的action和接收广播的action相同，接受者才能接受这个广播。

#广播有什么用
-----------------
其实，在什么是广播的第一句就已经说明了广播有什么用了。对了，笼统一点讲就是用来传输数据的。具体一点说就是：     
1. 实现了不同的程序之间的数据传输与共享，因为只要是和发送广播的action相同的接受者都能接受这个广播。典型的应用就是android自带的短信，电话等等广播，只要我们实现了他们的action的广播，那么我们就能接收他们的数据了，以便做出一些处理。比如说拦截系统短信，拦截骚扰电话等等 
2. 起到了一个通知的作用，比如在service中要通知主程序，更新主程序的UI等。因为service是没有界面的，所以不能直接获得主程序中的控件，这样我们就只能在主程序中实现一个广播接受者专门用来接受service发过来的数据和通知了。

#实现广播
------------
现在我们就来实现一个简单的广播程序。Android提供了两种注册广播接受者的形式，分别是在程序中动态注册和在xml中指定。他们之间的区别就是作用的范围不同，程序动态注册的接收者只在程序运行过程中有效，而在xml注册的接收者不管你的程序有没有启动有会起作用。
首先介绍在程序中动态注册的方式。

###动态注册方式
我们在程序中设置了三个按钮，分别是“注册广播”，“取消注册”和“发送广播”。然后每个按钮设置点击事件来完成广播的演示。最简单的项目的建立过程和按钮事件的建立我再这里就不罗嗦了，不会的可以下载下面的DEMO源码查看。直接看三个按钮的实现方式。
首先是注册广播的按钮事件代码：
```
private ReceiveBroadCast receiveBroadCast;  //广播实例
 
public class RegisteLinster implements OnClickListener
{
        @Override
        public void onClick(View view)
        {
            // 注册广播接收
            receiveBroadCast = new ReceiveBroadCast();
            IntentFilter filter = new IntentFilter();
            filter.addAction(flag);    //只有持有相同的action的接受者才能接收此广播
            registerReceiver(receiveBroadCast, filter);
        }
}
 
public class ReceiveBroadCast extends BroadcastReceiver
{
 
        @Override
        public void onReceive(Context context, Intent intent)
        {
            //得到广播中得到的数据，并显示出来
            String message = intent.getStringExtra("data");
            txtShow.setText(message);
        }
 
}
```
首先我们实现了一个ReceiveBroadCast 类，它继承了BroadcastReceiver并实现了其中的onReceive方法，这样当这个广播被接收的时候就会执行这个方法。注意我们在注册广播的时候使用了filter.addAction方法添加了一个过滤器。如果没有这一句，就相当于广播电台没有告诉咱们收音机用户接收的频率，就不好收听这个广播了。

再来看看如何取消注册，是的程序不再接收这个类型的广播了。
```
public class UnregisteLinster implements OnClickListener
{
 
        @Override
        public void onClick(View arg0)
        {
            unregisterReceiver(receiveBroadCast);
        }
}
```
怎么样？是不是超级简单的啊，就是将我们上面的那个广播类的实例传进去就行了。现在注册，取消注册都好了，就剩下如何发送了。看代码：
```
public class SendBroadCastListener implements OnClickListener
{
        @Override
        public void onClick(View arg0)
        {
            Intent intent = new Intent();  //Itent就是我们要发送的内容
            intent.putExtra("data", "this is data from broadcast "+Calendar.getInstance().get(Calendar.SECOND));  
            intent.setAction(flag);   //设置你这个广播的action，只有和这个action一样的接受者才能接受者才能接收广播
            sendBroadcast(intent);   //发送广播
        }
}
```
每一句都注释了的，就不要我再讲了吧。一看就明白了。现在，运行程序看看效果吧。先注册一下，然后每发送一次广播上面的文字就会变化一次，表明已经接收到了广播了。按取消注册后你可以发现再按发送按钮已经接收不到广播了。


###配置文件方式
配置和动态注册的区别在上面已经说了，这种方式适合你的程序需要长期的监测某个广播的情形，比如监测用户的短信。注册方式比较简单，相当于上面的代码只要接收的那部分就行了。不过要注意的是通过配置文件这种方式注册广播需要在单独的一个类中继承BroadReceiver，内部类是没有用的。所以我们新建了一个broadCastReceiveByXml类并继承了BroadReceive。代码如下：
```
public class broadCastReceiveByXml extends BroadcastReceiver
{
 
    @Override
    public void onReceive(Context arg0, Intent arg1)
    {
        Log.d("qlf", "broadcast receive by xml");    //因为不在主UI下，不好使用控件，所以我们这里打印到LOG里面查看效果
    }
 
}
```
然后在AndroidManifest中的<activity></activity>节点之后我们添加一下代码：
```
<receiver android:name="com.qlf.broadCast.broadCastReceiveByXml">
    <intent-filter>
        <action android:name="com.qlf.broadCastFlag">
        </action>
    </intent-filter>
</receiver>
```
receiver中的android：name就是我们在程序中的那个接收广播的类。下面的intent-filter和我们讲到的功能类似，而这个action就是上面的那个flag啦。现在我们运行程序，发现同样可以实现上面的功能。

除了使用我们自己发送广播，android也内置了许多广播。比如我们上面提到的来了消息的时候android会发送一个action名为`android.provider.Telephony.SMS_RECEIVED`的广播，这个时候如果我们想要接受这个广播只要将配置文件中的那个action设置为上面这个字符串就能接收到消息信息了。android包括了许多其他的广播action，有兴趣的同学到网上搜搜就有了。这里就不再举例了。
