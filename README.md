## 前言
在团队Android项目开发过程中，难免会出现一些比较不容易发现，但是又比较低级的bug。而且因为每个开发人员的编码习惯不同，写出的代码也会有差异。为了保证团队开发中代码的规范以及尽量避免低级bug，我们往往需要一些工具来进行严格的检查。下面将介绍在项目中用到的四种插件 **lint**、**findBugs**、**PMD**、 **CheckStyles** 的功能和使用方式，以及如何将多个插件整合在一起，在方便使用的同时尽量做到与项目工程解耦。

## 一、lint
Lint是Android Studio提供的一个代码检测工具，通过它开发者不用运行或者写测试代码，就可以发现和纠正问题，优化代码结构。每个被检测到的问题，都会生成一条描述信息并指明相应的严重性级别，当然这个严重性级别我们也可以自己设置的。

### 检测范围
+ 潜在的bug
+ 可优化的代码
+ 安全性
+ 性能
+ 可用性
+ 可访问性
+ 国际化

 ![](https://github.com/yushiwo/images/blob/master/code%20quality/lint/lint.png?raw=true)

### 插件安装
Android Studio自带，无需安装。

### 插件使用
##### 通过Gradle运行lint
在工程的根目录下运行相应的gradle task。 
 
+ Windows  

```
gradle lint
```

+ Linux 或者 MAC

```
./gradlew lint
```
当运行上面的命令执行完后，就会在项目目录/app/build/outputs/lint-results-debug.html生成相应的文件，可用浏览器打开查看。

##### 手动运行lint
有时我们可能只针对某个文件或者某个目录进行检测，这时使用gradle的方式就比较麻烦了，所以Android Studio提供给我们手动运行lint的方式。  

* 在AS的工程下选择module、目录或者文件  
* 右键选择**Analyze > Inspect Code.**  
* 此时会出现一个选择“指定检测范围”的dialog  
![](https://github.com/yushiwo/images/blob/master/code%20quality/lint/specify_inspection_scope_2x.png?raw=true)

* 配置完成后，点击**OK**按钮，进行检测。检测结果如下图所示，左边是检测类型的树形结构，右边则展示详细的信息。  
![](https://github.com/yushiwo/images/blob/master/code%20quality/lint/inspectandfix_2x.png?raw=true)

**注：**详细的使用，请看官网文档 [Improve Your Code with Lint](https://developer.android.com/studio/write/lint.html#manuallyRunInspections)

## 二、findBugs
FindBugs是一个Java静态分析工具，用来检查类或者jar文件，用来发现可能的问题。检测完成之后会生成一份详细的报告，借助这份报告可以找到潜在的bug，比如NullPointException，特定的资源没有关闭，查询数据库没有调用Cursor.close()等

### 检测范围
* 常见代码错误，序列化错误
* 可能导致错误的代码，如空指针引用
* 国际化相关问题：如错误的字符串转换
* 可能受到的恶意攻击，如访问权限修饰符的定义等
* 多线程的正确性：如多线程编程时常见的同步，线程调度问题。
* 运行时性能问题：如由变量定义，方法调用导致的代码低效问题

### 插件安装
在Android Studio中选择**Preferences -> Plugins**，输入查找findBugs进行插件安装。  
![](https://github.com/yushiwo/images/blob/master/code%20quality/findbugs/findbugs.png?raw=true)

### 插件使用
在build.gradle文件中，按照下面步骤进行设置：  
1. 添加plugin ``` apply plugin:'findbugs' ```  
2. 定义任务，指定输出格式

```
task findbugs(type: FindBugs, dependsOn: "assembleDebug") {
    ignoreFailures = true
    effort = "max"
    reportLevel = "high"
    excludeFilter = new File("$configDir/findbugs/findbugs-filter.xml")
    classes = files("${project.rootDir}/app/build/intermediates/classes")

    source 'src'
    include '**/*.java'
    exclude '**/gen/**'

    reports {
        xml.enabled = false
        html.enabled = true
        xml {
            destination "$reportsDir/findbugs/findbugs.xml"
        }
        html {
            destination "$reportsDir/findbugs/findbugs.html"
        }
    }

    classpath = files()
}
```
这里要注意因为findBugs是检查class文件，所以在定义task的时候，我们是**dependsOn: "assembleDebug"**，确保运行findbugs的task能够成功检测。

通过**gradle findbugs**方式，在工程目录下运行命令，检测完成后，会在制定的目录下生成报告文档。文档支持xml和html两种格式，本文设置的是html格式，可以直接用浏览器打开。  
当然，和lint一样，findBugs也支持手动检测的方式。在工程里，右键 **FindBugs -> （选择检测的范围）**。检测完之后，底部工具栏会跳到**FindBugs-IEDA**下，如图所示。
![](https://github.com/yushiwo/images/blob/master/code%20quality/findbugs/fingbugs_result.png?raw=true)

## 三、PMD
PMD是一个很有用的工具，它跟Findbugs类似，但是它不是检测字节码，它是直接检测源代码。它使用静态分析来发现错误。为什么要将它们同时使用呢？因为它们的检测方法不同，可以取到互补的作用。

### 检测范围
* 可能的bug——空的try/catch/finally/switch块。
* 无用代码(Dead code)：无用的本地变量，方法参数和私有方法。
* 空的if/while语句。
* 过度复杂的表达式——不必要的if语句，本来可以用while循环但是却用了for循环。
* 可优化的代码：浪费性能的String/StringBuffer的使用。

### 插件安装
同样可以通过AS的plugin进行安装，推荐安装QAPlug-PMD。
![](https://github.com/yushiwo/images/blob/master/code%20quality/PMD/PMD.png?raw=true)

### 插件使用
在build.gradle文件中进行如下配置

* 导入Plugin：```apply plugin: 'pmd' ```
* Task配置

```
task pmd(type: Pmd) {
    ignoreFailures = false
    ruleSetFiles = files("$configDir/pmd/pmd-ruleset.xml")
    ruleSets = []

    source 'src'
    include '**/*.java'
    exclude '**/gen/**'
    exclude 'androidTest/**'
    exclude 'test/**'

    reports {
        xml.enabled = false
        html.enabled = true
        xml {
            destination "$reportsDir/pmd/pmd.xml"
        }
        html {
            destination "$reportsDir/pmd/pmd.html"
        }
    }
}
```

## 四、CheckStyles
这个工具用来自动检测java源码。启动之后，可以按照制定的规则对java源码进行检查，被将所有的不符合规范的地方生成报告通知给你。

### 检测范围
* 注解
* javadoc注释
* 命名规范
* 文件头
* 导入包规范
* 尺寸设置
* 空格
* 正则表达式
* 修饰符
* 代码块
* 编码问题
* 类设计问题
* 重复、度量以及一些杂项

**总而言之，是一些代码规范问题!!**

### 插件安装
通过AS的Plugin进行安装
![](https://github.com/yushiwo/images/blob/master/code%20quality/checkstyle/checkStyle.png?raw=true)

### 插件使用
* 导入Plugin

```
apply plugin: 'checkstyle'
```

* 设置CheckStyle的版本

```
checkstyle {
    toolVersion '6.1.1'
    showViolations true
}
```

 * 配置任务  
 
```
task checkstyle(type: Checkstyle) {
    configFile file("$configDir/checkstyle/k12_checkstyle.xml")
    configProperties.checkstyleSuppressionsPath = file("$configDir/checkstyle/suppressions.xml").absolutePath
    source 'src'
    include '**/*.java'
    exclude '**/gen/**'
    exclude 'androidTest/**'
    exclude 'test/**'
    ignoreFailures true
    classpath = files()
}
```

## 五、插件的接入与使用

### 检测范围
lint、PMD、findBugs和CheckStyle检测范围之和。

### 插件安装
* 下载整合插件的文件包，和app工程目录同级放置。
* 在app的build.gradle文件导入整合插件脚本

```
apply from: '../config/quality.gradle'
```

### 插件使用
* 修改**quality.gradle** 的appDir字段，设置检测的工程目录

```
// 应用目录名称
def appDir = "app-k12"
```
* 进入工程根目录，运行**gradle check**，检测完成后，会在**build/reports**下生成相应的检测报告文件。当然也可以按照每个插件的使用方式单独使用。

## 参考文章
1. [How to improve quality and syntax of your Android code](http://vincentbrison.com/2014/07/19/how-to-improve-quality-and-syntax-of-your-android-code/)  
2. [Improve Your Code with Lint](https://developer.android.com/studio/write/lint.html)  
3. [Android 进阶之工具的使用 Findbugs](http://coderlin.coding.me/2016/06/22/Android-%E8%BF%9B%E9%98%B6%E4%B9%8B%E5%B7%A5%E5%85%B7%E7%9A%84%E4%BD%BF%E7%94%A8-Findbugs/)  
4. [Android静态代码检测](http://lcodecorex.github.io/2016/08/01/Android%E9%9D%99%E6%80%81%E4%BB%A3%E7%A0%81%E6%A3%80%E6%9F%A520160720/)  
5. [使用 CheckStyle 检查代码](http://gudong.name/2016/04/07/checkstyle.html)  
6. [详解CheckStyle的检查规则（共138条规则）](http://blog.csdn.net/yang1982_0907/article/details/18086693)  
7. [用 PMD 铲除 bug](http://www.ibm.com/developerworks/cn/java/j-pmd/)