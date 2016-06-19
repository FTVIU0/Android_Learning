`转载`
  
> 转自 [基础总结篇之一：Activity生命周期](http://blog.csdn.net/liuhe688/article/details/6733407) 有部分修改

我们来看一下这一张经典的生命周期流程图：

![Activity生命周期](http://hi.csdn.net/attachment/201109/1/0_1314838777He6C.gif)

相信不少朋友也已经看过这个流程图了，也基本了解了Activity生命周期的几个过程，我们就来说一说这几个过程。

1. 启动Activity：系统会先调用onCreate方法，然后调用onStart方法，最后调用onResume，Activity进入运行状态。

2. 当前Activity被其他Activity覆盖其上或被锁屏：系统会调用onPause方法，暂停当前Activity的执行。

3. 当前Activity由被覆盖状态回到前台或解锁屏：系统会调用onResume方法，再次进入运行状态。

4. 当前Activity转到新的Activity界面或按Home键回到主屏，自身退居后台：系统会先调用onPause方法，然后调用onStop方法，进入停滞状态。

5. 用户后退回到此Activity：系统会先调用onRestart方法，然后调用onStart方法，最后调用onResume方法，再次进入运行状态。

6. 当前Activity处于被覆盖状态或者后台不可见状态，即第2步和第4步，系统内存不足，杀死当前Activity，而后用户退回当前Activity：再次调用onCreate方法、onStart方法、onResume方法，进入运行状态。

7. 用户退出当前Activity：系统先调用onPause方法，然后调用onStop方法，最后调用onDestory方法，结束当前Activity。

但是知道这些还不够，我们必须亲自试验一下才能深刻体会，融会贯通。
下面我们就结合实例，来演示一下生命周期的几个过程的详细情况。我们新建一个名为lifecycle的项目，创建一个名为LifeCycleActivity的Activity，如下：

    package com.scott.lifecycle;

    import android.app.Activity;
    import android.content.Context;
    import android.content.Intent;
    import android.os.Bundle;
    import android.util.Log;
    import android.view.View;
    import android.widget.Button;

    public class LifeCycleActivity extends Activity {
	
	private static final String TAG = "LifeCycleActivity";
	private Context context = this;
	private int param = 1;
	
	//Activity创建时被调用
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.i(TAG, "onCreate called.");
        
        setContentView(R.layout.lifecycle);
        
        Button btn = (Button) findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
			@Override
			public void onClick(View v) {
				Intent intent = new Intent(context, TargetActivity.class);
				startActivity(intent);
			}
		});
    }
    
    //Activity创建或者从后台重新回到前台时被调用
    @Override
    protected void onStart() {
    	super.onStart();
    	Log.i(TAG, "onStart called.");
    }
    
    //Activity从后台重新回到前台时被调用
    @Override
    protected void onRestart() {
    	super.onRestart();
    	Log.i(TAG, "onRestart called.");
    }
    
    //Activity创建或者从被覆盖、后台重新回到前台时被调用
    @Override
    protected void onResume() {
    	super.onResume();
    	Log.i(TAG, "onResume called.");
    }
    
    //Activity窗口获得或失去焦点时被调用,在onResume之后或onPause之后
    /*@Override
    public void onWindowFocusChanged(boolean hasFocus) {
    	super.onWindowFocusChanged(hasFocus);
    	Log.i(TAG, "onWindowFocusChanged called.");
    }*/
    
    //Activity被覆盖到下面或者锁屏时被调用
    @Override
    protected void onPause() {
    	super.onPause();
    	Log.i(TAG, "onPause called.");
    	//有可能在执行完onPause或onStop后,系统资源紧张将Activity杀死,所以有必要在此保存持久数据
    }
    
    //退出当前Activity或者跳转到新Activity时被调用
    @Override
    protected void onStop() {
    	super.onStop();
    	Log.i(TAG, "onStop called.");	
    }
    
    //退出当前Activity时被调用,调用之后Activity就结束了
    @Override
    protected void onDestroy() {
    	super.onDestroy();
    	Log.i(TAG, "onDestory called.");
    }
    
    /**
     * Activity被系统杀死时被调用.
     * 例如:屏幕方向改变时,Activity被销毁再重建;当前Activity处于后台,系统资源紧张将其杀死.
     * 另外,当跳转到其他Activity或者按Home键回到主屏时该方法也会被调用,系统是为了保存当前View组件的状态.
     * 在onPause之前被调用.
     */
	@Override
	protected void onSaveInstanceState(Bundle outState) {
		outState.putInt("param", param);
		Log.i(TAG, "onSaveInstanceState called. put param: " + param);
		super.onSaveInstanceState(outState);
	}
	
	/**
	 * Activity被系统杀死后再重建时被调用.
	 * 例如:屏幕方向改变时,Activity被销毁再重建;当前Activity处于后台,系统资源紧张将其杀死,用户又启动该Activity.
	 * 这两种情况下onRestoreInstanceState都会被调用,在onStart之后.
	 */
	@Override
	protected void onRestoreInstanceState(Bundle savedInstanceState) {
		param = savedInstanceState.getInt("param");
		Log.i(TAG, "onRestoreInstanceState called. get param: " + param);
		super.onRestoreInstanceState(savedInstanceState);
	}
    }

大家注意到，除了几个常见的方法外，我们还添加了**onWindowFocusChanged、onSaveInstanceState、onRestoreInstanceState**方法：

*1.onWindowFocusChanged方法：*

在Activity窗口获得或失去焦点时被调用，例如创建时首次呈现在用户面前；当前Activity被其他Activity覆盖；当前Activity转到其他Activity或按Home键回到主屏，自身退居后台；用户退出当前Activity。以上几种情况都会调用onWindowFocusChanged，并且当Activity被创建时是在onResume之后被调用，当Activity被覆盖或者退居后台或者当前Activity退出时，它是在onPause之后被调用，如图所示：

![](http://i.imgur.com/jafaIeQ.png)

这个方法在某种场合下还是很有用的，例如程序启动时想要获取视特定视图组件的尺寸大小，在onCreate中可能无法取到，因为窗口Window对象还没创建完成，这个时候我们就需要在**onWindowFocusChanged**里获取；如果大家已经看过我写的[Android动画之Frame Animation](http://blog.csdn.net/liuhe688/article/details/6657776)这篇文章就会知道，当时试图在onCreate里加载frame动画失败的原因就是因为窗口Window对象没有初始化完成，所以最后我将加载动画的代码放到了onWindowFocusChanged中，问题迎刃而解。不过大家也许会有疑惑，为什么我在代码里将它注释掉了，因为对当前Activity每一个操作都有它的执行log，我担心这会影响到整个流程的清晰度，所以将它注掉，大家只要了解它应用的场合和执行的顺序就可以了。

*2.onSaveInstanceState：*

(1) 在Activity被覆盖或退居后台之后，系统资源不足将其杀死，此方法会被调用；

(2) 在用户改变屏幕方向时，此方法会被调用；

(3) 在当前Activity跳转到其他Activity或者按Home键回到主屏，自身退居后台时，此方法会被调用。第一种情况我们无法保证什么时候发生，系统根据资源紧张程度去调度；第二种是屏幕翻转方向时，系统先销毁当前的Activity，然后再重建一个新的，调用此方法时，我们可以保存一些临时数据；第三种情况系统调用此方法是为了保存当前窗口各个View组件的状态。onSaveInstanceState的调用顺序是在onPause之前。

*3.onRestoreInstanceState：*

(1) 在Activity被覆盖或退居后台之后，系统资源不足将其杀死，然后用户又回到了此Activity，此方法会被调用；

(2) 在用户改变屏幕方向时，重建的过程中，此方法会被调用。我们可以重写此方法，以便可以恢复一些临时数据。onRestoreInstanceState的调用顺序是在onStart之后。

以上着重介绍了三个相对陌生方法之后，下面我们就来操作一下这个Activity，看看它的生命周期到底是个什么样的过程：

1.启动Activity：

![](http://i.imgur.com/mdZTZSw.png)

在系统调用了onCreate和onStart之后，调用了onResume，自此，Activity进入了运行状态。

2.跳转到其他Activity，或按下Home键回到主屏：

![](http://i.imgur.com/t3lLFTz.png)

我们看到，此时onSaveInstanceState方法在onPause之前被调用了，并且注意，退居后台时，onPause后onStop相继被调用。

3.从后台回到前台：

![](http://i.imgur.com/fdE5o0w.png)

当从后台会到前台时，系统先调用onRestart方法，然后调用onStart方法，最后调用onResume方法，Activity又进入了运行状态。

4.修改TargetActivity在AndroidManifest.xml中的配置，将android:theme属性设置为**@android:style/Theme.Dialog**，然后再点击LifeCycleActivity中的按钮，跳转行为就变为了TargetActivity覆盖到LifeCycleActivity之上了，此时调用的方法为：

![](http://i.imgur.com/cDvpRGB.png)

注意还有一种情况就是，我们点击按钮，只是按下锁屏键，执行的效果也是如上。
我们注意到，此时LifeCycleActivity的OnPause方法被调用，并没有调用onStop方法，因为此时的LifeCycleActivity没有退居后台，只是被覆盖或被锁屏；onSaveInstanceState会在onPause之前被调用。

5.按回退键使LifeCycleActivity从被覆盖回到前面，或者按解锁键解锁屏幕：

![](http://i.imgur.com/J9CbFuS.png)

此时只有onResume方法被调用，直接再次进入运行状态。

6.退出：

![](http://i.imgur.com/H8nae6S.png)

最后onDestory方法被调用，标志着LifeCycleActivity的终结。

大家似乎注意到，在所有的过程中，并没有onRestoreInstanceState的出现，这个并不奇怪，因为之前我们就说过，onRestoreInstanceState只有在杀死不在前台的Activity之后用户回到此Activity，或者用户改变屏幕方向的这两个重建过程中被调用。我们要演示第一种情况比较困难，我们可以结合第二种情况演示一下具体过程。顺便也向大家讲解一下屏幕方向改变的应对策略。

首先介绍一下关于Activity屏幕方向的相关知识。

我们可以为一个Activity指定一个特定的方向，指定之后即使转动屏幕方向，显示方向也不会跟着改变：

1.指定为竖屏：在AndroidManifest.xml中对指定的Activity设置**android:screenOrientation="portrait"**，或者在onCreate方法中指定：

    setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);	//竖屏

2.指定为横屏：在AndroidManifest.xml中对指定的Activity设置**android:screenOrientation="landscape"**，或者在onCreate方法中指定：

    setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);	//横屏

为应用中的Activity设置特定的方向是经常用到的办法，可以为我们省去不少不必要的麻烦。不过，我们今天讲的是屏幕方向改变时的生命周期，所以我们并不采用固定屏幕方向这种办法。

下面我们就结合实例讲解一下屏幕转换的生命周期，我们新建一个Activity命名为OrientationActivity，如下：

    package com.scott.lifecycle;

    import android.app.Activity;
    import android.content.res.Configuration;
    import android.os.Bundle;
    import android.util.Log;

    public class OrientationActivity extends Activity {
	
	private static final String TAG = "OrientationActivity";
	private int param = 1;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.orientation_portrait);
		Log.i(TAG, "onCreate called.");
	}
	
	@Override
	protected void onStart() {
		super.onStart();
		Log.i(TAG, "onStart called.");
	}
	
	@Override
	protected void onRestart() {
		super.onRestart();
		Log.i(TAG, "onRestart called.");
	}
	
	@Override
	protected void onResume() {
		super.onResume();
		Log.i(TAG, "onResume called.");
	}
	
	@Override
	protected void onPause() {
		super.onPause();
		Log.i(TAG, "onPause called.");
	}
	
	@Override
	protected void onStop() {
		super.onStop();
		Log.i(TAG, "onStop called.");
	}
	
	@Override
	protected void onDestroy() {
		super.onDestroy();
		Log.i(TAG, "onDestory called.");
	}

	@Override
	protected void onSaveInstanceState(Bundle outState) {
		outState.putInt("param", param);
		Log.i(TAG, "onSaveInstanceState called. put param: " + param);
		super.onSaveInstanceState(outState);
	}
	
	@Override
	protected void onRestoreInstanceState(Bundle savedInstanceState) {
		param = savedInstanceState.getInt("param");
		Log.i(TAG, "onRestoreInstanceState called. get param: " + param);
		super.onRestoreInstanceState(savedInstanceState);
	}
	
	//当指定了android:configChanges="orientation"后,方向改变时onConfigurationChanged被调用
	@Override
	public void onConfigurationChanged(Configuration newConfig) {
		super.onConfigurationChanged(newConfig);
		Log.i(TAG, "onConfigurationChanged called.");
		switch (newConfig.orientation) {
		case Configuration.ORIENTATION_PORTRAIT:
			setContentView(R.layout.orientation_portrait);
			break;
		case Configuration.ORIENTATION_LANDSCAPE:
			setContentView(R.layout.orientation_landscape);
			break;
		}
	}
    }

首先我们需要进入“Settings->Display”中，将“Auto-rotate Screen”一项选中，表明可以自动根据方向旋转屏幕，然后我们就可以测试流程了，当我们旋转屏幕时，我们发现系统会先将当前Activity销毁，然后重建一个新的：

![](http://i.imgur.com/DZw2K0m.png)

系统先是调用onSaveInstanceState方法，我们保存了一个临时参数到Bundle对象里面，然后当Activity重建之后我们又成功的取出了这个参数。

为了避免这样销毁重建的过程，我们需要在AndroidMainfest.xml中对OrientationActivity对应的<activity>配置**android:configChanges="orientation"**，然后我们再测试一下，我试着做了四次的旋转，打印如下：

![](http://i.imgur.com/kChrN3A.png)

可以看到，每次旋转方向时，只有onConfigurationChanged方法被调用，没有了销毁重建的过程。

以下是需要注意的几点：

1. 如果<activity>配置了android:screenOrientation属性，则会使android:configChanges="orientation"失效。


2. 模拟器与真机差别很大：模拟器中如果不配置android:configChanges属性或配置值为orientation，切到横屏执行一次销毁->重建，切到竖屏执行两次。真机均为一次。模拟器中如果配置android:configChanges="orientation|keyboardHidden"（如果是Android4.0，则是"orientation|keyboardHidden|screenSize"），切竖屏执行一次onConfigurationChanged，切横屏执行两次。真机均为一次。


Activity的生命周期与程序的健壮性有着密不可分的关系，希望朋友们能够认真体会、熟练应用。





