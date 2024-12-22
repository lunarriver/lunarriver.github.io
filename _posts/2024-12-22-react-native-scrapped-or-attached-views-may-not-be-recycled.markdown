---
layout: post
title:  "React Native升级到新架构的问题：Scrapped or attached views may not be recycled"
date:   2024-12-22 12:24:00 +0800
---

RN项目升级到新架构(Turbo Module & Fabric Componenet)后，在Android端运行，页面回退时发生崩溃。崩溃信息如下：

```
java.lang.IllegalArgumentException: Scrapped or attached views may not be recycled. isScrap:false isAttached:true androidx.viewpager2.widget.ViewPager2$RecyclerViewImpl{2b04b0c VFED..... ......ID 0,0-820,194 #6}, adapter:com.reactnativepagerview.ViewPagerAdapter@aa9598c, layout:androidx.viewpager2.widget.ViewPager2$LinearLayoutManagerImpl@335cc55, context:com.facebook.react.uimanager.ThemedReactContext@349b319
	at androidx.recyclerview.widget.RecyclerView$Recycler.recycleViewHolderInternal(RecyclerView.java:7071)
	at androidx.recyclerview.widget.RecyclerView.removeAnimatingView(RecyclerView.java:1597)
	at androidx.recyclerview.widget.RecyclerView$ItemAnimatorRestoreListener.onAnimationFinished(RecyclerView.java:13655)
	at androidx.recyclerview.widget.RecyclerView$ItemAnimator.dispatchAnimationFinished(RecyclerView.java:14157)
	at androidx.recyclerview.widget.SimpleItemAnimator.dispatchRemoveFinished(SimpleItemAnimator.java:286)
	at androidx.recyclerview.widget.DefaultItemAnimator$4.onAnimationEnd(DefaultItemAnimator.java:216)
	at android.view.ViewPropertyAnimator$AnimatorEventListener.onAnimationEnd(ViewPropertyAnimator.java:1115)
	at android.animation.Animator$AnimatorListener.onAnimationEnd(Animator.java:600)
	at android.animation.ValueAnimator.endAnimation(ValueAnimator.java:1333)
	at android.animation.ValueAnimator.cancel(ValueAnimator.java:1218)
	at android.view.ViewPropertyAnimator.cancel(ViewPropertyAnimator.java:428)
	at androidx.recyclerview.widget.DefaultItemAnimator.cancelAll(DefaultItemAnimator.java:649)
	at androidx.recyclerview.widget.DefaultItemAnimator.endAnimations(DefaultItemAnimator.java:639)
	at androidx.recyclerview.widget.RecyclerView.onDetachedFromWindow(RecyclerView.java:3424)
	at android.view.View.dispatchDetachedFromWindow(View.java:21345)
	at android.view.ViewGroup.dispatchDetachedFromWindow(ViewGroup.java:3957)
	at android.view.ViewGroup.dispatchDetachedFromWindow(ViewGroup.java:3949)
	at android.view.ViewGroup.dispatchDetachedFromWindow(ViewGroup.java:3949)
	at android.view.ViewGroup.clearDisappearingChildren(ViewGroup.java:7084)
	at android.view.ViewGroup.dispatchDetachedFromWindow(ViewGroup.java:3951)
	at android.view.ViewGroup.clearDisappearingChildren(ViewGroup.java:7084)
	at android.view.ViewGroup.dispatchDetachedFromWindow(ViewGroup.java:3951)
	at android.view.ViewGroup.clearDisappearingChildren(ViewGroup.java:7084)
	at android.view.ViewGroup.dispatchDetachedFromWindow(ViewGroup.java:3951)
	at android.view.ViewGroup.dispatchDetachedFromWindow(ViewGroup.java:3949)
	at android.view.ViewGroup.endViewTransition(ViewGroup.java:7195)
	at com.swmansion.rnscreens.ScreenStack.endViewTransition(ScreenStack.kt:63)
	at androidx.fragment.app.DefaultSpecialEffectsController$startAnimations$3.onAnimationEnd$lambda$0(DefaultSpecialEffectsController.kt:272)
	at androidx.fragment.app.DefaultSpecialEffectsController$startAnimations$3.$r8$lambda$ZytGaoJ8By--dVBbT6zTRvs0sIA(Unknown Source:0)
	at androidx.fragment.app.DefaultSpecialEffectsController$startAnimations$3$$ExternalSyntheticLambda0.run(D8$$SyntheticClass:0)
	at android.os.Handler.handleCallback(Handler.java:942)
	at android.os.Handler.dispatchMessage(Handler.java:99)
	at android.os.Looper.loopOnce(Looper.java:201)
	at android.os.Looper.loop(Looper.java:288)
```

其中，除android.\*和androidx.\*外，与项目相关的类是ScreenStack。

ScreenStack是react-native-screens库中的类。尝试将其升级至最新版本，未能解决问题。

在react-native-screens的github issues中的找到相关问题：[Error while going back with Top Tabs · Issue #2461 · software-mansion/react-native-screens · GitHub](https://github.com/software-mansion/react-native-screens/issues/2461)

中，有人提及在使用到Material Top Tabs Navigator、TopNavBackButton的页面中回退，会产生同样的crash，但是我的RN项目并没有使用到此组件。

有人提及把gradle.properties中的newArchEnabled置为false时，问题解决。但是，我的RN项目需要升级到新的架构。

再次回到崩溃信息。定位到最顶层的IllegalArgumentException异常，它由RecyclerView的recycleViewHolderInternal()方法抛出：

```
void recycleViewHolderInternal(ViewHolder holder) {
	if (holder.isScrap() || holder.itemView.getParent() != null) {
		throw new IllegalArgumentException(
				"Scrapped or attached views may not be recycled. isScrap:"
						+ holder.isScrap() + " isAttached:"
						+ (holder.itemView.getParent() != null) + exceptionLabel());
	}
	...
}

String exceptionLabel() {
	return " " + super.toString()
			+ ", adapter:" + mAdapter
			+ ", layout:" + mLayout
			+ ", context:" + getContext();
}
```

异常信息提示的是：RecyclerView是回收ViewHolder时发现其关联的itemView没有被回收。

导致异常抛出的有关RecyclerView的最上层调用是DefaultItemAnimator，与致使崩溃的页面回退操作相联系，猜测崩溃产生的可能原因是：

- ScreenStack是一个RN页面的栈，这个栈的实现基于RecyclerView(ViewPager2$RecyclerViewImpl)，它的适配器实现则是ViewPagerAdapter;

- 页面回退时，作为RecyclerView的itemView的一个页面从适配器中被剔除时，执行过度动画(联想到ScreenStack的endTransition()方法名中的'Transition')；

定位到ScreenStack的endViewTransition()方法，发现其为ViewGroup的方法，且有与之相对应的startViewTransition()方法，尝试将这两个方法的方法体注释掉。不能解决问题，仍然崩溃，只是崩溃信息不同。

这两个方法是ViewGroup的方法，它们可能是View机制的一部分。

在确认RN项目中react-native-screens的库是最新版本后，把注意力转移后js代码中触发页面回退的调用：

```
props.navigation.goBack();
```

该函数是@react-navigation库的API。在react-navigation库的[doc页](https://reactnavigation.org/docs/getting-started/)中，可以看到它是基于react-navite-screens实现的。

猜测可能是由于react-navigation对ScreenStack的相关操作诱发了崩溃。

尝试将@react-navigation库的相关依赖的版本升级至最新版本，包括@react-navigation/native、@react-navigation/stack、@react-navigation/native-stack等。问题未得到解决，仍崩溃。

尝试按照最新版本的@react-navigation改写项目中的页面导航部分。主要是Stack的创建方式和navigation通过useNavigation()获取方式存在差异。问题未得到解决，仍崩溃。

尝试将项目中所有使用到的依赖均升级到最新版本。问题未得到解决，仍崩溃。

尝试阅读@react-navigation库、react-native-screens库的源码，以期进行断点排查，被@react-navigation库的typescript类型声明的复杂度劝退。

此时，已然失去了解决问题的明确路径和突破口。但是，页面导航应该是UI框架的基本特性，react-native团队在做新架构时不应该会忽视这样的问题。而在react-native-screens的github issue页中，关于此问题的回复也没有那么多。这就说明，这样的一个问题不是普遍的，一定是我的RN项目中的某些方面没有搞对。

回到我的RN项目本身，尝试在每个页面进行回退操作，进而发现，并不是每个页面在回退时都会崩溃。结合着崩溃信息，推测产生崩溃的可能原因是：

- 页面回退时，视图层次结构中的View逐个被销毁，ViewGroup的endTransition()方法可能是这个过程中的一环；

- 崩溃信息中的RecyclerView相关信息则可能指向的是，该页面存在着某些RN组件，它的原生层面的实现是基于RecyclerView(ViewPager2)，而这类组件是销毁时产生了错误；

基于上述推测，对各个页面的崩溃情况进行分类，并对视图层次结构中的RN组件进行识别，找到原生层面基于RecyclerView的RN组件。

最终，问题指向了react-native-pager-view这个库。某些并不是每次回退都崩溃的页面，其崩溃与否取决于基于该库实现的组件是否挂载则证实了这一点。

查看该库在npm上的[说明](https://npmmirror.com/package/react-native-pager-view?version=6.6.1)，作者更新了一个版本，尝试进行升级。问题得到解决，不再崩溃。

最后，查看实现Material Top Tabs Navigator组件的@react-navigation/material-top-tabs库的[doc](https://reactnavigation.org/docs/material-top-tab-navigator)，该库果然也是基于react-native-pager-view实现的。