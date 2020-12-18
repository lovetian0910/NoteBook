# Android桌面挂件踩坑日记

## RemoteViewsService复用问题
AppWidget开发中如果需要使用“Collection View”，如ListView、GridView等，就需要用到RemoteViewsService来更新UI，在RemoteViewsService中会创建RemoteViewsFactory作为Collection View的Adapter。

但我遇到的问题是如果同时存在多个同类型的GridView，想使用相同的RemoteViewsService来更新UI，所有GridView的内容都变成一样的了。

Debug后发现RemoteViewsFactory只创建了一次，导致所有的GridView都共用了同一份数据。

看了一下源码，发现了问题所在：

```kotlin
@Override
public IBinder onBind(Intent intent) {
    synchronized (sLock) {
        Intent.FilterComparison fc = new Intent.FilterComparison(intent);
        RemoteViewsFactory factory = null;
        boolean isCreated = false;
        if (!sRemoteViewFactories.containsKey(fc)) {
            factory = onGetViewFactory(intent);
            sRemoteViewFactories.put(fc, factory);
            factory.onCreate();
            isCreated = false;
        } else {
            factory = sRemoteViewFactories.get(fc);
            isCreated = true;
        }
        return new RemoteViewsFactoryAdapter(factory, isCreated);
    }
}
```
可以看到这里是创建RemoteViewsFactory的关键代码，其中最主要的是会用Intent.FilterComparison为Key从HashMap里获取RemoteViewsFactory。

FilterComparison中重写了equals方法，可以看到主要调用了Intent的filterEquals方法。
```kotlin
@Override
public boolean equals(Object obj) {
    if (obj instanceof FilterComparison) {
        Intent other = ((FilterComparison)obj).mIntent;
        return mIntent.filterEquals(other);
    }
    return false;
}
```

接着来看filterEquals方法，里面判断了Action、Data、Type、Identifier、Package、Component、Categories是否相同来判断两个Intent是否相同。
```kotlin
public boolean filterEquals(Intent other) {
    if (other == null) {
        return false;
    }
    if (!Objects.equals(this.mAction, other.mAction)) return false;
    if (!Objects.equals(this.mData, other.mData)) return false;
    if (!Objects.equals(this.mType, other.mType)) return false;
    if (!Objects.equals(this.mIdentifier, other.mIdentifier)) return false;
    if (!Objects.equals(this.mPackage, other.mPackage)) return false;
    if (!Objects.equals(this.mComponent, other.mComponent)) return false;
    if (!Objects.equals(this.mCategories, other.mCategories)) return false;

    return true;
}
```
分析到这里我们的问题就很清晰了，解决方案就是要在更新不同View时创建不同的Intent。我这里采用了StackOverflow上的一个方法，使用setData来区分不同的Intent：
```kotlin
val intent = Intent(context, MatchWidgetTeamIconService::class.java).apply {
    data = Uri.parse(this.toUri(Intent.URI_INTENT_SCHEME))
}
```