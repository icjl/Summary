Intent IntentFilter
===

### 关系

- Intent: 根据特定的条件找到匹配的组件。
- IntentFilter: 为某个组件向系统注册一些特性，以便 Intent 找到对应组件。

从先后关系上看，IntentFilter 在前，任何一个组件必须先通过 IntentFilter 注册。Intent 在后，根据特定信息，找到之前以及注册过的组件。

### 源码分析
 
Intent 类源码路径：\frameworks\base\core\java\android\content\Intent.java

```java
public class Intent implements Parcelable, Cloneable {  
  
    private String mAction;              // action 值  
    private Uri mData;                   // uri  
    private String mType;                // MimeType  
    private String mPackage;             // 所在包名  
    private ComponentName mComponent;    // 组件信息  
    private int mFlags;                  // Flag 标志位  
    private HashSet<String> mCategories; // Category 值  
    private Bundle mExtras;              // 附加值信息  
    //...
}
```

IntentFilter类源码路径：\frameworks\base\core\java\android\content\IntentFilter.java

```java
public class IntentFilter implements Parcelable {  
    //...  
    // 保存了所有action字段的值  
    private final ArrayList<String> mActions;  
    // 保存了所有Category的值  
    private ArrayList<String> mCategories = null;  
    // 保存了所有Schema(模式)的值  
    private ArrayList<String> mDataSchemes = null;  
    // 保存了所有Authority字段的值  
    private ArrayList<AuthorityEntry> mDataAuthorities = null;  
    // 保存了所有Path的值  
    private ArrayList<PatternMatcher> mDataPaths = null;  
    // 保存了所有MimeType的值  
    private ArrayList<String> mDataTypes = null;  
    //...  
}
```

可见，Intent 中属性类型基本上都是单个类型的，而 IntentFilter 属性都是集合类型的。

### 匹配规则

##### 动作检测

清单文件中的 <intent-filter> 元素以 <action> 子元素列出动作，例如：

```xml
<intent-filter>
    <action android:name="com.example.project.SHOW_CURRENT" />
    <action android:name="com.example.project.SHOW_RECENT" />
</intent-filter>
```

一个过滤器必须至少包含一个 <action> 属性，否则将阻塞所有 Intent。

##### 种类检测

```xml
<intent-filter>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
</intent-filter>
```

如果 intent 对象中 category 字段为空，不管 IntentFilter 中有什么种类，总是通过匹配。
但是有个例外，Android 对待所有传递给 startActivity() 的隐式 intent 好像它们至少包含 android.intent.category.DEFAULT。
因此，Activity 想要接收隐式 intent 必须要在 intent 过滤器中包含 android.intent.category.DEFAULT。

##### 数据检测

类似的，清单文件中的<intent-filter>元素以<data>子元素列出数据，例如：

```xml
<intent-filter . . . >
    <data android:mimeType="video/mpeg" android:scheme="http" /> 
    <data android:mimeType="audio/mpeg" android:scheme="http" />
    . . .
</intent-filter>
```

每个 <data> 元素指定一个 URI 和数据类型（MIME类型）。它有四个属性 scheme、host、port、path 对应于 URI 的每个部分：
```
scheme://host:port/path

例：
content://com.example.project:200/folder/subfolder/etc
scheme: content
host: com.example.project
port: 200
path: folder/subfolder/etc

host 和 port 一起构成 URI  authority，如果 host 没有指定，port 也将被忽略。 

这四个属性都是可选的，但它们之间并不都是完全独立的。
要让 authority 有意义，scheme 必须也要指定。
要让 path 有意义，scheme 和 authority 也都必须要指定。
```

当比较 intent 对象和过滤器的 URI 时，仅仅比较过滤器中出现的 URI 属性。
例如，如果一个过滤器仅指定了 scheme，所有有此 scheme 的 URIs 都匹配过滤器；
如果一个过滤器指定了 scheme 和 authority，但没有指定 path，所有匹配 scheme 和 authority 的 URIs 都通过检测，而不管它们的 path；
如果四个属性都指定了，要都匹配才能算是匹配。然而，过滤器中的path可以包含通配符来要求匹配path中的一部分。

<data>元素的type属性指定数据的MIME类型。Intent对象和过滤器都可以用"*"通配符匹配子类型字段，例如"text/*"，"audio/*"表示任何子类型。

数据检测既要检测 URI，也要检测数据类型。规则如下：

- 一个Intent对象既不包含URI，也不包含数据类型：仅当过滤器也不指定任何URIs和数据类型时，才不能通过检测；否则都能通过。
- 一个Intent对象包含URI，但不包含数据类型：仅当过滤器也不指定数据类型，同时它们的URI匹配，才能通过检测。例如，mailto:和tel:都不指定实际数据。
- 一个Intent对象包含数据类型，但不包含URI：仅当过滤也只包含数据类型且与Intent相同，才通过检测。
- 一个Intent对象既包含URI，也包含数据类型（或数据类型能够从URI推断出）：数据类型部分，只有与过滤器中之一匹配才算通过；URI部分，它的URI要出现在过滤器中，或者它有content:或file: URI，又或者过滤器没有指定URI。换句话说，如果它的过滤器仅列出了数据类型，组件假定支持content:和file: 。

如果一个Intent能够通过不止一个活动或服务的过滤器，用户可能会被问那个组件被激活。如果没有目标找到，会产生一个异常。


##### 通用情况

上面最后一条规则表明组件能够从文件或内容提供者获取本地数据。因此，它们的过滤器仅列出数据类型且不必明确指出content:和file: scheme的名字。这是一种典型的情况，一个<data>元素像下面这样：
```xml
<data android:mimeType="image/*" />
```
告诉 Android 系统这个组件能够从内容提供者获取 image 数据并显示它。因为大部分可用数据由内容提供者（content provider）分发，过滤器指定一个数据类型但没有指定 URI 或许最通用。

另一种通用配置是过滤器指定一个scheme和一个数据类型。例如，一个<data>元素像下面这样：

```xml
<data android:scheme="http" android:type="video/*" />
```
告诉Android这个组件能够从网络获取视频数据并显示它。考虑，当用户点击一个web页面上的link，浏览器应用程序会做什么？它首先会试图去显示数据（如果link是一个HTML页面，就能显示）。如果它不能显示数据，它将把一个隐式Intent加到scheme和数据类型，去启动一个能够做此工作的活动。如果没有接收者，它将请求下载管理者去下载数据。这将在内容提供者的控制下完成，因此一个潜在的大活动池（他们的过滤器仅有数据类型）能够响应。

大部分应用程序能启动新的活动，而不引用任何特别的数据。活动有指定"android.intent.action.MAIN"的动作的过滤器，能够启动应用程序。如果它们出现在应用程序启动列表中，它们也指定"android.intent.category.LAUNCHER"种类：

```xml
<intent-filter . . . >
    <action android:name="code android.intent.action.MAIN" />
    <category android:name="code android.intent.category.LAUNCHER" />
</intent-filter>
```
