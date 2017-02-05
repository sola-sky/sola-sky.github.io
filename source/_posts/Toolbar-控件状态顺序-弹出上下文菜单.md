---
title: Toolbar 控件状态顺序 弹出上下文菜单
date: 2016-05-23 18:59:40
tags: [toolbar selector 上下文菜单]
---

##Toolbar

运用到Toolbar的时候，主题必须是NoActionBar的。

##selector 控件状态顺序

selector中的顺序很重要，当状态都满足时，会选择顺序在前面的进行显示，比如
``` xml
<item android:drawable="@drawable/bottomnavbar_rewind_sel" android:state_pressed="true" />
<item android:drawable="@drawable/bottomnavbar_rewind_sel" android:state_enabled="false" />
```

**当状态同时满足`state_pressed="true"`和`state_enabled="false"`时，显示顺序在前面的设置，这里`state_pressed="true"`在前面，所以显示其设置。反之若`state_enable="false"`在前面，则显示其设置。**

## 上下文菜单

当给listview设置弹出菜单时，这时要求listview里item项里的子控件不能设置onClickListenter或者设置可以点击等属性，不然会弹不出菜单。但是非要设置可点击，监听时，这时需要给该子控件也加设置上下文菜单（`setOnCreateContextMenuListener()`），里面什么都不需要写，交给listview设置上下文菜单时的监听处理就行了。