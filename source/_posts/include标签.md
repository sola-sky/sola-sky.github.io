---
title: include标签
date: 2016-05-25 09:52:05
tags:
---

# include标签

在&lt;include&gt;标签当中，我们是可以覆写所有layout属性的，即include中指定的layout属性将会覆盖掉layout="@layout/..."xml中指定的layout属性。 
如果我们想要在&lt;include&gt;标签当中覆写layout属性，必须要将layout_width和layout_height这两个属性也进行覆写，否则覆写效果将不会生效。
还有就是当给include加id时，会覆写其引入的xml布局中的id，如果布局直接是一个控件，则该控件的id就变成了include设置的id，直接通过`findViewById()`传入include的id就能找到该控件。如果布局是一个ViewGroup，则该id就变变成了include设置的id，想找其下的子view时得通过`findViewById(include的id).findViewById(子控件的id)`这种方法找到子控件。
查了android官网上面有一句**Overrides the ID given to the root view in the included layout.**