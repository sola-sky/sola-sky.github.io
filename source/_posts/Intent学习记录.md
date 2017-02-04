---
title: Intent学习记录
date: 2016-05-22 14:30:28
tags: intent
catagories: android-intent
---
> **[以下内容来自刘明渊的博客](http://blog.csdn.net/vanpersie_9987)**



- 显示Intent



android5.0之后规定必须显示启动Service。



- 隐式Intent



当你**隐式地**启动一个activity时，intent会根据其中的内容，匹配其他组件中manifest文件的intent-filter，启动符合条件的组件，并把intent中参数传递过去，如果有多个intent-filter满足条件，那么系统将会弹出一个对话框，由用户自己决定启动那个组件。

**intent-filter**是manifest文件内部的一个标签，该标签描述了组件具备身体特性，如果组件未配置intent-filter，那么该组件只能显示启动了。



### 创建Intent对象

intent中包含了目标组件需满足的特性。Intent中应包含以下信息：



- Component name:



目标组件的名字。对于显示启动，这是不可缺省的，可以使用Intent的构造函数传入组件名字，也可以调用`setComponent()`,`setClass()`,`setClassName()`这些方法传入组件名。当然若是隐式启动，这些便是可选的了，但是相对的intent中应包含其他的信息（**action、category、data**）。



- Action：



是一个可以指明目标组件行为的字符串。**action**很大程度上决定了category和data中应该传入的信息；也可以在自己的应用程序组件中指定action，以便让其他应用程序启动自己的组件。对应于action中字符串，不建议使用硬编码的形式，而应该在所属组件的类中设置为常量。

常见的action有：ACTION_VIEW（一般用于向用户展示一些信息），ACTION_SEND（一般需要向其附带一些发送信息，这些信息由目标activity决定发送给谁，如社交类app）。

可以将action作为参数传入intent构造函数中，也可以用setAction()。

如需要自定义组件的action，应该用应用的包名作为前缀如"com.apical."。



- Data



创建一个Intent时，除了为data指定URI之外，还应该指定data的MIME类型。比如启动一个展示图片的activity不能用于播放音乐的，所以就需要将MIMI指定为"image/\*"；有些时候data能从URI中就能推断出MIME的类型，比如当一个URI的schema是"content://"时，表明该URI指定了设备内部的一个文件并由ContentProvider管理着，系统可以根据该文件推断data的MIME类型。

可以调用`setData()`设置URI，调用`setType()`设置MIME类型，或者调用`setDataAndType()`同时设置URI和MIME类型。

**注意如果需要同时设置URI和MIME类型时必须调用setDataAndType()，而不能分别调用setData()和setType()，因为setData()时会将setType()的内容置空，而调用setType()时也是一样的。**



- Category



是一个字符串，表示目标组件的附加信息。

**CATEGORY_BROWSEABLE:**表示目标activity可以被网页上的某个链接启动，比如图片activity或e-mail信息的activity。

可以将category参数通过addCategory()方法传入。



- 还有**Extras**和**Flags**



extras可以携带附加信息，以键值对的形式存储，而flags则指导系统以何种方式启动activity，是否将activity放在该应用的任务栈中，等等。



### 注意

* 若系统中没有满足隐式Intent的目标组件，应用将崩溃，所以最好先判断再启动`startActivity()`，可以用`intent.resolveActivity(getPackageManager())`进行非空判断。对了还有就是`Intent.createChooser()`可以保证每次都弹出选择列表。

+ 还有就是如组件需要被隐式启动，则必须在manifest中为其配置**CATEGORY_DEFAULT**。

+ `<action android:name="android.intent.action.MAIN />"` `<category android:name="android.intent.category.LAUNCHER" />`这两个应该成对出现。

+ 使用Pending Intent：PendingIntent是一个包装Intent的类，主要用于实现Intent的延迟操作，主要的应用场合有：1. 包装一个Notification的启动Intent  2. 包装一个App Widget的Intent（按Home键启动的activity）  3. 包装一个延迟启动的activity（如AlarmManager）。  通过PendingIntent启动的activity无需使用`startActivity()`，而是使用`PendingIntent.getActivity(), getService(), getBroadcast()`

+ action标签可以定义多个，intent只要匹配其中一个就行，而category也可以定义多个，但是intent中定义的每一个category都必须在intent-filter中，否则不行（注意intent-filter中的category可能会比intent中定义的多，但只要intent中是其子集就行）。

+ 通过startActivity()方法隐式启动intent中，将被自动添加一个CATEGORY_DEFAULT的category标签，所以若希望activity能被隐式启动，则需要在intent-filter中添加一个android.intent.category.DEFAULT的category标签。

+ 匹配Data的时候，每个data标签都能设置mimeType和URI结果，其中URI可分为四部分：scheme，host，port和path，如content://com.apical.cloud:520/folder/subfolder/etc,其中content为scheme，com.apical.cloud为host，520为port，后面的就是path了。但是每一部分在标签中都不是必须定义的，但存在一个线性依赖：若scheme未指定，则host被忽略。若host未指定，port被忽略，若scheme和host均未被指定，则path被忽略。



在intent中添加的data只需要匹配一部分intent-filter中的data（URI匹配）：



- 若filter只定义了scheme，则intent的data定义的URI中只要包含了相同的scheme，就能匹配；

- 若filter只定义了scheme和host，则intent的data定义的URI中只要包含了相同的scheme和host，就能匹配；

- 若filter只定义了scheme、host和port，则intent的data定义的URI中只要包含了相同的scheme、host和port，就能匹配；



intent-filter匹配intent的data中URI和mimeType类型的规则如下：



1. 如果intent-filter中未指定data，则未添加data的intent可以匹配；

2. 如果intent-filter中指定了URI，但未指定mimeType，则按照上一段的规则匹配（intent中也应未指定mimeType）；

3. 如果intent-filter中指定了mimeType，而未指定URI，则可以匹配intent中指定了相同mimeType，而未指定URI的组件；

4. 如果intent-filter中同时指定了mimeType和URI，则：

- intent中添加的mimeType只要能匹配上intent-filter中的某一个mimeType，就能匹配上mimeType部分；

- intent中添加的URI则按照上一段中的URI匹配规则，就能该匹配上URI部分；**特别地：若intent中的URI的scheme 指定为content: 或者 file:，那么即便intent-filter中未定义URI，也能匹配成功。换句话说：包含content: 或者 file: 的URI总是能匹配上只定义了mimeType的intent-filter。**

