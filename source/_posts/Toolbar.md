---
title: Toolbar
date: 2016-06-07 15:39:16
tags:
---

### Toolbar
Toolbar默认只显示一个标题文本，默认情况下该标题文本会使用AndroidManifest中当前Activity节点下label标签所对应的文本，如果当前Activity节点下没有label标签则查找上级节点application中的label标签文本显示，这点与ActionBar类似。  
Toolbar继承于ViewGroup，因此可以像普通控件那样使用它。**使用时必须注意Activity的theme必须设置为NoActionBar的，否则将出错。**
调用`setSupportActionBar()`将Toolbar作为ActionBar来对待，这时你需要通过onCreateOptionsMenu和onOptionsItemSelected方法来联系菜单和监听菜单事件

**上述是将Toolbar当做ActionBar来使用的情况，实际上我们也可以将其独立作为一款控件来使用，这时不需要用setSupportActionBar设置它为ActionBar就行了**，这时你可以通过`inflateMenu()`方法来加载菜单文件，并通过`setOnMenuItemClickListener()`方法监听。**切记如果是用了setSupportActionBar时，这两个方法无效，不会产生想要的扩展自定义的菜单和事件监听**。单作为普通控件时，弹出二级菜单时，不能像ActionBar那样便捷地通过Theme来修改，但Toolbar提供了一个setPopupTheme方法和对应的popupTheme属性来设置弹出菜单的样式。

**要使用ActionMode,必须将Toolbar作为ActionBar，即设置setSupportActionBar()**

#### ActionMode

An action mode's lifecycle is as follows:

- onCreateActionMode(ActionMode, Menu) once on initial creation
- onPrepareActionMode(ActionMode, Menu) after creation and any time the ActionMode is invalidated
- onActionItemClicked(ActionMode, MenuItem) any time a contextual action button is clicked
- onDestroyActionMode(ActionMode) when the action mode is closed

###### onCreateActionMode

Called when action mode is first created. The menu supplied will be used to generate action buttons for the action mode.
true if the action mode should be created, false if entering this mode should be aborted.

###### onDestroyActionMode

Called when an action mode is about to be exited and destroyed.

###### onPrepareActionMode

Called to refresh an action mode's action menu whenever it is invalidated.
true if the menu or action mode was updated, false otherwise.

### Menu

- actionViewClass属性可以用来指定item控件的类型
- menu.findItem().getActionView()可以获得控件