

> 该篇为我学习GLib 和 GTK 的笔记，只是记录一些知识点和写法，若碰巧浏览此篇，尽量不要以此篇为准！

## GLib相关介绍

### GObject

------



#### GObject简单声明定义

GObject实现中， 类是两个结构体的组合，一个是**实例结构体**（保存所有对象私有数据），另一个是**类结构体**（保存所有对象共享的数据），声明如下：

```c
// dlist.h 实现一个列表类
#include <glib-object.h>

#define PM_TYPE_DLIST (pm_dlist_get_type())

typedef struct _PMDListNode PMDListNode;
struct  _PMDListNode {
        PMDListNode *prev;
        PMDListNode *next;
        void *data;
};

typedef struct _PMDList PMDList;
struct  _PMDList {
        GObject parent_instance;
        PMDListNode *head;
        PMDListNode *tail;
};

typedef struct _PMDListClass PMDListClass;

struct _PMDListClass {
        GObjectClass parent_class;
};

GType pm_dlist_get_type (void);
```

```c
// dlist.cpp
#include "dlist.h"
// 对 pm_dlist_get_type 生成实现，返回类对象
// #arg_1: 类名 	#arg_2: 成员函数命名前缀	#arg_3: 父类型
G_DEFINE_TYPE (PMDList, pm_dlist, G_TYPE_OBJECT);
static void pm_dlist_init (PMDList *self)
{
        g_printf ("\t实例结构体初始化！\n");
        self->head = NULL;
        self->tail = NULL;
}

static void pm_dlist_class_init (PMDListClass *klass)
{
        g_printf ("类结构体初始化!\n");
}
```

GObject具有功能：

- 基于引用计数的内存管理
- 对象的构造函数与析构函数
- 可设置对象属性的 set/get 函数
- 易于使用的信号机制



继承GObject基类的实例化代码：

```c
PMDList *dlist; /* 类的实例化，产生对象 */
dlist = g_object_new (PM_TYPE_DLIST, NULL); /* 创建对象的一个实例 并将其引用计数 +1 */
g_object_unref (dlist); /* 将对象的实例引用计数 -1，并检测对象的实例的引用计数是否为 0，若为 0，那么便释放对象的实例的存储空间。 */
dlist = g_object_new (PM_TYPE_DLIST, NULL); /* 再创建对象的一个实例 */
```



GObject子类化完整的过程：

> ① 在 .h 文件中包含 glib-object.h；
> ② 在 .h 文件中构建实例结构体与类结构体，并分别将 GObject 类的实例结构体与类结构体置于成员之首；
> ③ 在 .h 文件中定义 P_TYPE_T 宏，并声明 p_t_get_type 函数；
> ④ 在 .c 文件中调用 G_DEFINE_TYPE 宏产生类型注册代码。

声明的简单范例，参考地址： https://blog.csdn.net/knowledgebao/article/details/82418046



#### GObject 设置属性



#### GObject 的继承

```c
// kb-Son.h
#include "kb-Parent.h"

typedef struct _KbSon KbSon;
struct _KbSon {
        KbParent parent;	// 继承父实例属性
};

typedef struct _KbSonClass KbSonClass;
struct _KParentClass {
        KbParentClass parent_class;	// 继承父类属性
};
```

```c
// kb-Son.c
...
G_DEFINE_TYPE(KbSon, kb_son, KB_TYPE_Parent);	// GType 设置成父类 其他代码一样
...
```



继承常用的宏（其中P表示项目名称 	T表示类名称	PTPrivate表示私有数据结构体）：

```c
#define P_TYPE_T (p_t_get_type())	// 仅在使用 g_object_new 进行对象实例化的时候使用一次，用于向 GObject 库的类型系统注册 PT 类
#define P_T(obj) (G_TYPE_CHECK_INSTANCE_CAST ((obj), P_TYPE_T, PT))	// 用于将 obj 对象的类型强制转换为 P_T 类的实例结构体类型
#define P_IS_T(obj) G_TYPE_CHECK_INSTANCE_TYPE((obj), P_TYPE_T)) // 用于判断 obj 对象的类型是否为 P_T 类的实例结构体类型
#define P_T_CLASS(klass) (G_TYPE_CHECK_CLASS_CAST ((klass), P_TYPE_T, PTClass))// 用于将 kclass 类结构体得类型强制转换为 P_T 类的类结构体类型
#define P_IS_T_CLASS(klass) (G_TYPE_CHECK_CLASS_TYPE ((klass), P_TYPE_T))	// 用于判断 klass 类结构体的类型是否为 P_T 类的类结构体类型
#define P_T_GET_CLASS(obj) (G_TYPE_INSTANCE_GET_CLASS((obj), P_TYPE_T, PTClass))	// 获取 obj 对象对应的类结构体类型
#define P_T_GET_PRIVATE(obj) (G_TYPE_INSTANCE_GET_PRIVATE((obj), P_TYPE_T, PTPrivate))	// 获取 obj 对象对应的私有数据
```



接口常用的宏（其中 P 表示项目名称	T表示类名称	I是接口的缩写）

```c
#define P_TYPE_IT (p_t_get_type())	// 仅在接口实现时使用一次，用于向 GObject 库的类型系统注册 PIT 接口
#define P_IT(obj) (G_TYPE_CHECK_INSTANCE_CAST((obj), P_TYPE_IT, P_IT))	// 用于将 obj 对象的类型强制转换为 P_IT 接口的实例结构体类型
#define P_IS_IT(obj) (G_TYPE_CHECK_INSTANCE_TYPE((obj), P_TYPE_IT))	// 用于判断 obj 对象是否为 P_IT接口的实例结构体类型
#define P_IT_GET_INTERFACE(obj) (G_TYPE_INSTANCE_GET_INTERFACE ((obj), P_TYPE_IT, P_IT))	// 获取 obj 对象对应的 P_IT 接口的类结构体类型
```



#### GObject 的信号使用

```c
// 新建信号
guint g_signal_new (const gchar		*signal_name,
                    GType				   itype,
                    GSignalFlags	signal_flags,
                    guint           class_offset,
                    GSignalAccumulator	 		accumulator,
                    gpointer		 			accu_data,
                    GSignalCMarshaller  		c_marshaller,
                    GType               		return_type,
                    guint               		n_params,
                    ...);

// 连接信号和回调函数
gulong g_signal_connect_data (gpointer	instance, const gchar	*detailed_signal,
                              GCallback	  			c_handler,
                              gpointer		  			 data,
                              GClosureNotify	 destroy_data,
                              GConnectFlags	  	 connect_flags);

// 发射信号
void g_signal_emit_by_name (gpointer	instance, const gchar	*detailed_signal, ...);
```





## GTK

**GtkApplication**

用于处理GTK+初始化、应用程序唯一性、会话管理，通过导出操作和菜单提供一些基本的脚本能力和桌面shell集成，并管理一个顶级窗口列表，其生命周期自动绑定到应用程序的生命周期。



**GtkWindow**

一个 **GtkWindow** 是一个可以包含其他控件的顶级窗口，窗口通常在桌面系统下具有样式。并且允许用户放缩、移动或者关闭窗体等操作。



```c
// 显示窗体
void gtk_window_present (GtkWindow* window)
```

