## QML 学习之路

工作了三年，一直用**QGui** 开发比较简单的程序，之所以开始接触**QML**是因为现在客户端已经逐渐转入**JavaScript** 那套开发模式，这绝非危言耸听，并且推出 **webassembly** 后，我认为客户端和前端会逐渐走向**UI**逻辑由**JavaScript**编写，高性能的运算和操作系统设备会给 **C/C++/Rust** 编写的 **webassembly**  模块执行，这可能客户端真正意义上的强后端分离。虽然很不想承认，自己过去三年所学的技术正在犹如日出日落般推出主流的客户端开发，可能仅仅只能维持老的项目和嵌入式方向， 但客户端和前端能做的事情变得越来越多，深度也在不加深，以前人们一直以为后端比前端更有技术含量，从现在来看，前端程序员逐渐百花齐放起来，他们实现得很多效果，我自认拍马也赶不上。 为了以后能有碗饭吃，我决定给自己挖一口井，学习**QML** 并从**QML**逐渐过度到前端学习上。好了，现在开始吧！



## 一、什么是 **QML**

可能你对 **HTML** 比较熟悉，其实 **QML** 要做的事情和HTML 有些类似 —— 描述用户界面。**QML** 用于描述用户界面元素的形状和行为。用户界面能够使用 **JavaScript** 来提供修饰，或者增加更加复杂的逻辑。其结构层次：子元素从父元素上继承坐标系统，它的 **x，y** 坐标总是对应于它的父元素坐标系统。



首先使用一个简单的 **QML** 例子：

```qml
import QtQuick 2.12
import QtQuick.Window 2.12

Window {
    width: 640
    height: 480
    visible: true
    title: qsTr("测试窗口")
    Text {
        id: helloWord
        text: qsTr("hello world")
        anchors.centerIn: parent
        font.bold: true;
        font.pointSize: 20
    }
}
```

- **import** 声明导入一个指定模块，**JavaScript**资源和组件目录，写法如下：

  ```javascript
  import <ModuleIdentifier> <Version.Number> [as <Qualifier>]
  ```

- 每个 **.qml**  文件有且只有一个根元素！整个结构为属性结构

- 任何在 **QML** 文档中的元素可以通过 属性**id** 进行访问，在同一个 .qml 文件里，**id** 是唯一的 （注意: ***一个元素id应该只在当前文档中被引用。QML提供了动态作用域的机制，后加载的文档会覆盖之前加载文档的元素id号，这样就可以引用已加载并且没有被覆盖的元素id，这有点类似创建全局变量。但不幸的是这样的代码阅读性很差。目前这个还没有办法解决这个问题，所以你使用这个机制的时候最好仔细一些甚至不要使用这种机制。如果你想向文档外提供元素的调用，你可以在根元素上使用属性导出的方式来提供***。）



## **二、元素属性** （Properties）

属性是可以赋予静态值或绑定到动态表达式的对象的属性。属性的值可以由其他对象读取。通常，它也可以由另一个对象修改，除非特定的 **QML** 类型明确禁止对特定属性执行此操作。

| 属性值的类型 |                           **解释**                           |
| :----------: | :----------------------------------------------------------: |
|    静态值    |                   不依赖其他属性的值的常量                   |
|  绑定表达式  | 描述属性与其他属性关系的JavaScript表达式。此表达式中的变量称为属性的依赖项。<br/>QML引擎强制执行属性及其依赖项之间的关系。当任何依赖项的值发生更改时，QML引擎会自动重新计算绑定表达式，并将新结果赋给该属性。 |



**QML** 提供基础组件例如：**Item**、**Text**等都有自己特有的 **Property**， 并且我们可以添加组件的额外属性，其写法如下：

```qml
	property type name : value
```

其中 **type** 指 **value** 的类型，**qml** 支持的值类型如下：

|    类型     |           说明           |
| :---------: | :----------------------: |
|    bool     |        true/false        |
|   double    |       双精度浮点型       |
| enumeration |        命名枚举值        |
|     int     |           整形           |
|    list     |       QML 对象列表       |
|    real     |      等同于 double       |
|   string    |          字符串          |
|     url     |        资源定位器        |
|     var     | 通用类型，根据值进行推到 |
|    date     |           日期           |
|    point    |          (x, y)          |
|    rect     |  (x, y, width, height)   |
|    size     |     (width, height)      |

除了添加组件自定义属性， 还可以使用 **alias** 关键字用于转发一个属性或者转发一个组件对象的属性(例如： 组件想暴露子对象的属性)，一个属性别名不需要类型，它会根据引用的属性进行推导。语法如下：

```qml
property alias proprerty_name : another.proprety_name 
property alias new_name : old_name 
```



## 三、基本元素 （Basic Element）

元素可以被分为可视化元素和非可视元素。一个可视化元素（例如矩形框Rectangle）有着几何形状并且可以在屏幕上显示。一个非可视化元素（例如计时器Timer）提供了常用的功能，通常用于操作可视化元素。

Item 是所有可视化元素的基础对象，所有其他的可视化元素都继承自Item。 它自身不会有任何绘制操作，但是定义了所有可视化元素共有的属性：

| Group(分组)                  | Properties（属性）                                           |      |
| ---------------------------- | ------------------------------------------------------------ | ---- |
| Geometry (几何属性)          | x， y （坐标）定义了元素左上角的位置，width，height(长和宽) 定义元素的显示范围，z(堆叠次序)定义元素之间的重叠顺序 |      |
| Layout handling（布局操作）  | anchors（锚定），包括左（left），右（right），上（top），下（bottom），水平与垂直居中（vertical center，horizontal center），与margins（间距）一起定义了元素与其它元素之间的位置关系。 |      |
| Key handling（按键操作）     | 附加属性 key (按键) 和 keyNavigation（按键定位）属性来控制按键操作，处理输入焦点（focus）可用操作 |      |
| Transformation（转换）       | 缩放（scale）和rotate（旋转）转换，通用的x,y,z属性列表转换（transform），旋转基点设置（transformOrigin）。 |      |
| Visual （可视化）            | 不透明度（opacity）控制透明度，visible（是否可见）控制元素是否显示，clip（裁剪）用来限制元素边界的绘制，smooth（平滑）用来提高渲染质量。 |      |
| State definition（状态定义） | states（状态列表属性）提供了元素当前所支持的状态列表，当前属性的改变也可以使用transitions（转变）属性列表来定义状态转变动画。 |      |



### 四、定位元素 （Positioning Element）

分为：**Column**、**Row**、**Grid（栅格布局）**、**Flow（流式）**



### 五、布局元素 （Layout Items）

一般 **QML** 使用 **anchors**（锚）对元素进行布局。其优先级比几何变化（例如：**x**、**y**、**width**、**height**）高， 这可以理解成 **QGui**

中布局优先级比 **width** 和 **height** 高。

