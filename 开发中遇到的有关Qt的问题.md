## 开发中遇到的有关Qt的问题

1. 向下面的代码对某 a 列设置伸展属性后， 再设置 b 列的列宽为 0 时导致无法隐藏这一列，b 列会显示一个固定宽度

   ```c++
   ui->tablewidget->horizontalHeader()->setSectionResizeModel(1, QHeaderView::Stretch);
   ```

   

2. **QStyle** 类中 **drawPrimitive** 方法解读

   ```c++
   /**
     * @brief drawPrimitive 绘制原生控件
     * @param element 要绘制原生控件类型， 例如: 按钮 文本 复选框等
     * @param option 渲染空间的参数, 例如: element 是 PE_PanelButtonCommand —— 按钮, 那么在渲染的时候需要知道要渲染位置，尺寸，颜色等等, 这些参数都由 option 指定
     * @param painter 实际执行渲染的设备
     * @param widget The widget argument is optional and may contain a widget that may aid in drawing the primitive element.
     */
   void QStyle::drawPrimitive(QStyle::PrimitiveElement element, const QStyleOption *option, QPainter *painter, const QWidget *widget = nullptr) const
   ```

3. **QLineEdit** 的 **placeholderText()** 是指当 QLineEdit 内容为空时, 会显示编辑框的提示内容 , 用户输入那内容后就消

   

4. 当 **QTableWidget** 某单元格内容很长显示不下时, **QTableWidgetItem** 可以设置 **ToolTip()** , 这样当光标移入该单元格时会弹出提示文本

   

5. 获取 **A** 控件的 **QStyle** 对象, 调用该对象的**drawControl** 绘制带有 **Qss** 样式的控件, 需要对 **A** 控件先设置 **Qss** 属性否则调用 **drawControl** 无法的让 **Qss** 样式生效.

   

6. **QGraphicsView** 相当于一个窗户， **QGraphicsScene** 相当于窗外的景色。如果不做其他设置**QGraphicsScene** 的中心(注意此处中心不是 **QGraphicsScene** 的原点)处于**QGraphicsView** 区域的中心， **x**轴向右， **y**轴向下。而这个原点可以的在 **QGraphicsView** 的坐标系上被记录。**QGraphicsView::setSceneRect** 可以调整 **QGraphicsScene** 的坐标范围，例如**QGraphicsView::setSceneRect(-10, 10,  200,  200)** 表示 Scene 的坐标范围为**(-10, -10, 200, 200)**。但是**QGraphicsView** 中心最初还是和 **QGraphicsScene** 的中心重合



7. **QMetaObject::invokeMethod** 非常好用, 它可以省去 **connect** 这个动作，却可以像 **connect** 一样的效果调用的目标对象的槽函数或者是被 **Q_INVOKABLE** 标记的函数; 但请注意, 用 **QMetaObject::invokeMethod** 去调用槽函数时, 槽函数并不能通过 **QObject::sender()** 获得到调用对象，返回的地址是 **0x0**。

   

8. **QBoxLayout** 对某个**QWidget**对象或者 **QLayout** 设置 **stretch** (权值) 为0, 其效果是在主动拉伸的时候, 这一列不做垂直方向或者水平方向的长度变化。当对某个 QWidget 设置了(**sizePolicy**)垂直策略或者水平策略为 **Fixed**, 则该水平方向上不会拉伸(该优先级比父类布局的权值优先级更高)

   

9. 在使用布局时候，子控件谨慎使用 **FixedSize** 等类似方法， 例如: 当子控件放进 **H**水平布局，**H**布局放进一个**V**垂直布局并且给这个**H**布局的权值(**stretch**)设为**0**，想让 **V** 布局在垂直方向拉伸时，**H**布局的高度不变。但当对子控件设置 **FixedSize** 时， 并且 **FixedSize** 设置的高度超过了 **V** 布局分配给H布局的高度时, 此时 **V** 布局对**H**布局设置的权值被打破，并且**H**布局会跟着**V**布局一起做拉伸。

   

10. **QCompleter**  配置 **QLineEdit**  , **QCombobox** 实现动态输入匹配功能, 其匹配模式看以参看枚举变量 **Qt::MatchFlag** 。QCompleter 默认维护一个 QListView 控件用于显示补全的信息，其源码为:

    ```C++
    /*!
    	Returns the popup used to display completions.
    
    	\sa setPopup()
    */
    QAbstractItemView *QCompleter::popup() const
    {
    	Q_D(const QCompleter);
    #if QT_CONFIG(listview)
    	if (!d->popup && completionMode() != QCompleter::InlineCompletion) {
    		QListView *listView = new QListView;
    		listView->setEditTriggers(QAbstractItemView::NoEditTriggers);
    		listView->setHorizontalScrollBarPolicy(Qt::ScrollBarAlwaysOff);
    		listView->setSelectionBehavior(QAbstractItemView::SelectRows);
    		listView->setSelectionMode(QAbstractItemView::SingleSelection);
    		listView->setModelColumn(d->column);
    		QCompleter *that = const_cast<QCompleter*>(this);
    		that->setPopup(listView);
    	}
    #endif // QT_CONFIG(listview)
    	return d->popup;
    }
    ```

    所以在使用的时候， 可以使用 **Qss** 对 **QCompleter**  持有的 **QListView** 样式生效。因为 **QListView** 在 默认声明的时候并没有将父类指针指向 **QLineEdit** 或 其他对象(实际给**QLineEdit** 赋予一个父指针会使补全功能失效 )。可以给 **QLineEdit** 统一设置同一个 **objectname**， 然后使用全局**Qss** 设置，让所有的补全编辑框保持统一风格



11. **QLineEdit** 配合 **QAction** 可以实现编辑框的内置图标，  **QAction** 还可以配合 **QMenu**、**QToolButton**等控件

    

12. **QOpenGLWdget** 和 **QGLWidget** 的透明度问题相关讨论地址: https://blog.csdn.net/qq21497936/article/details/94635225

    

13. 通过重写 **QGraphicsItem** 的 **type()** 方法， 自定义 **item** 类型值, 枚举值从 **QGraphicsItem::UserType** 的基础上递增。配合 **qgraphicsitem_cast** 进行基类**item**向子类自定义的**item**转换。使用例子如下:

    ```c++
    QList<QGraphicsItem *> items = scene->items();
    foreach (QGraphicsItem *item, items) {
        if (item->type() == QGraphicsRectItem::Type) {  // 矩形
            QGraphicsRectItem *rect = qgraphicsitem_cast<QGraphicsRectItem*>(item);
            // 访问 QGraphicsRectItem 的成员
        } else if (item->type() == QGraphicsLineItem::Type) {  // 直线
            QGraphicsLineItem *line = qgraphicsitem_cast<QGraphicsLineItem*>(item);
            // 访问 QGraphicsLineItem 的成员
        } else if (item->type() == QGraphicsProxyWidget::Type) {  // 代理 Widget
            QGraphicsProxyWidget *proxyWidget = qgraphicsitem_cast<QGraphicsProxyWidget*>(item);
            QLabel *label = qobject_cast<QLabel *>(proxyWidget->widget());
            // 访问 QLabel 的成员
        } else if (item->type() == CustomItem::Type) {  // 自定义 Item
            CustomItem *customItem = qgraphicsitem_cast<CustomItem*>(item);
            // 访问 CustomItem 的成员
        } else {
            // 其他类型 item
        }
    }
    ```

    

14. **QHeaderView** 可以通过 **setSectionResizeMode** 设置表头调节大小的属性

    ```C++
    enum QHeaderView::ResizeMode
    {
        QHeaderView::Interactive,  ///< 可以通过鼠标等对表头单元进行拖拽来调整宽度
        QHeaderView::Fixed, ///< 只能通过程序设置调整表头单元宽度
        QHeaderView::Stretch,	///< 表头单元格尺寸根据剩余下的宽度做尺寸调整 其他修改方式无效
        QHeaderView::ResizeToContents,	///< 根据表头里面的文本内容来调整表头宽度
    }
    ```

15. 宏定义 ’**#**‘ 和 ’**##**‘ 的使用说明: '**#**' 用于将代码转换成字符串; '**##**' 用于拼接代码; 使用范例:

    ```c++
    #include <iostream>
    
    #define PrintCode(x) #x
    #define INT_TYPE(x) int##x##_t
    
    int main(int argc, char *argv[])
    {
    	std::cout << PrintCode(int) << std::endl;	// 输出: int
    	INT_TYPE(64) a;
    	std::cout << sizeof(a) << std::endl;	// 输出 8 因为 a是 int64_t
    	return 0;
    }
    ```
    
    

16. Qss 选择想要选择多个同级的控件, 用逗号分割，例如:

    ```css
    QPushButton#button_1, QPushButton#button_2 { color:red; }
    ```

    
    
17. STL 容器在调用resize做内存扩充时, 会默认初始化 节点<T>对象，STL 源码如下:

    ```C++
    void resize(size_type __new_size)
    {
    	if (__new_size > size())
    	  _M_default_append(__new_size - size());
    	else if (__new_size < size())
    	  _M_erase_at_end(this->_M_impl._M_start + __new_size);
    }
    
    template<typename _Tp, typename _Alloc>
    void vector<_Tp, _Alloc>::_M_default_append(size_type __n)
    {
      if (__n != 0)
    	{
    	  if (size_type(this->_M_impl._M_end_of_storage
    					- this->_M_impl._M_finish) >= __n)
    		{
    		  _GLIBCXX_ASAN_ANNOTATE_GROW(__n);
    		  this->_M_impl._M_finish =
    			std::__uninitialized_default_n_a(this->_M_impl._M_finish,
    											 __n, _M_get_Tp_allocator());
    		  _GLIBCXX_ASAN_ANNOTATE_GREW(__n);
    		}
    	  else
    		{
    		  const size_type __len =
    			_M_check_len(__n, "vector::_M_default_append");
    		  const size_type __old_size = this->size();
    		  pointer __new_start(this->_M_allocate(__len));
    		  pointer __new_finish(__new_start);
    		  __try
    			{
    			  __new_finish
    				= std::__uninitialized_move_if_noexcept_a
    				(this->_M_impl._M_start, this->_M_impl._M_finish,
    				 __new_start, _M_get_Tp_allocator());
    			  __new_finish =
    				std::__uninitialized_default_n_a(__new_finish, __n,
    												 _M_get_Tp_allocator());
    			}
    		  __catch(...)
    			{
    			  std::_Destroy(__new_start, __new_finish,
    							_M_get_Tp_allocator());
    			  _M_deallocate(__new_start, __len);
    			  __throw_exception_again;
    			}
    		  _GLIBCXX_ASAN_ANNOTATE_REINIT;
    		  std::_Destroy(this->_M_impl._M_start, this->_M_impl._M_finish,
    						_M_get_Tp_allocator());
    		  _M_deallocate(this->_M_impl._M_start,
    						this->_M_impl._M_end_of_storage
    						- this->_M_impl._M_start);
    		  this->_M_impl._M_start = __new_start;
    		  this->_M_impl._M_finish = __new_finish;
    		  this->_M_impl._M_end_of_storage = __new_start + __len;
    		}
    	}
    }
    ```

    
    
18. **QPlainTextEdit** 在 **minimumHeight** 设置为 **0** 后， 此时将 **QPlainTextEdit** 放入到布局中，运行后用鼠标整个窗体进行缩小, **QPlainTextEdit** 在缩小到某种程度便会停止从而撑起整个布局， 并不会缩小到 **0px**



19. 在想声明成员变量都是同类型的结构体时，例如:

    ```C++
    struct T_A{
        int a = 1;
        int b = 2;
        int c = 3:
        ...
    };
    ```

    可以用结合数组和枚举的形式，例如：

    ```c++
    enum eIndex
    {
    	EINDEX_a = 0,
        EINDEX_b,
        EINDEX_c,
        ...
        EINDEX_size
    };
    
    int arriA[] = { 1, 2, 3 ...};
    // 取值
    arriA[EINDEX_a] = 10;
    arriA[EINDEX_b] = 10;
    arriA[EINDEX_c] = 10;
    
    // 这样的好处是当成员变量非常多的时候，可以使用 for 循环进行操作
    
    for (int i = 0; i < EINDEX_size; i++)
    {
        arriA[i] += 10; 
    }
    
    ```

    
    
20. 以前在使用 **Qt** 槽函数的时候经常遇到 **signal** 名称重复的情况，例如以前 **QNetworkReply** 在比较低的版本，其信号 **error** 和 其公有方法**error** 冲突。导致并不能直接使用 **functor**式的连接，然后可以使用 **static_cast** 进行一些的转换，如下：

    ```c++
    connect(networkReply, static_cast<void(QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error),
        [=](QNetworkReply::NetworkError code){ /* ... */ });
    ```

    偶然发现 **QGlobal.h** 提供 **QOverload** 来简化上面例子中的写法，代码如下：

    ```c++
    connect(networkReply, QOverload<QNetworkReply::NetworkError>::of(&QNetworkReply::error),
        [=](QNetworkReply::NetworkError code){ /* ... */ });
    ```

    相比于使用 **static_cast** 要理解晦涩难懂函数指针,  **QOverload** 则是隐藏了 函数指针的转换，对新手来说更容易理解，以下是 **QOverload** 的源码:

    ```c++
    template <typename... Args>
    struct QOverload : QConstOverload<Args...>, QNonConstOverload<Args...>
    {
    	using QConstOverload<Args...>::of;
    	using QConstOverload<Args...>::operator();
    	using QNonConstOverload<Args...>::of;
    	using QNonConstOverload<Args...>::operator();
    	
    	template <typename R>
    	Q_DECL_CONSTEXPR auto operator()(R (*ptr)(Args...)) const Q_DECL_NOTHROW -> decltype(ptr)
    	{ return ptr; }
    	
    	template <typename R>
    	static Q_DECL_CONSTEXPR auto of(R (*ptr)(Args...)) Q_DECL_NOTHROW -> decltype(ptr)
    	{ return ptr; }
    };
    ```

    
