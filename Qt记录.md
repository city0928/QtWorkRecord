# Qt记录

## Qt布局器管理内存

- 布局器的使用会管理所布局的窗口，注意窗口的二次析构问题；

## C++11

- decltype 自动表达式推导

```
int a = 10, b = 9;
declatype(a+b) c = 0;
```

- 深浅拷贝

```
//浅拷贝
类中的成员变量都是值类型 int、double、float等
在重写类的拷贝构造函数中都会进行浅拷贝（值的拷贝）

//深拷贝
在类中成员变量被声明为指针类型的，需要在拷贝构造函数中进行新的内存地址的拷贝；
如果只是简单的值拷贝则会使得指针指向同一份内存地址中；应该为该变量重新分配内存地址；
```

- 保存图片（opencv和QImage）

要保存不压缩的图片文件，是需要额外设置参数的，opencv 和 QImage 都是这样

```
// opencv:
    qDebug() << "start...";
    cv::Mat c = imread("BoardRef-0.tif", IMREAD_COLOR);
    qDebug() << "opencv large loaded.";
    // TIFF 编码压缩方式设置为不压缩：
    std::vector<int> params;// params 添加顺修不可反
    params.push_back(IMWRITE_TIFF_COMPRESSION); // TIFF 编码压缩方式设置
    params.push_back(1); // 不压缩
    imwrite("BoardRef-0-copy-large-opencv.tif", c, params); // 写文件时传入设置好的参数列表
    qDebug() << "uncompressed saved.";
    qDebug() << "done.";


// qimage:
    QImageReader::setAllocationLimit(4000);
    qDebug() << "start...";
    QImage qimage("BoardRef-B.tif");
    qDebug() << "qimage large loaded.";
    qimage.save("BoardRef-B-large-qimage-copy.tif", nullptr, 100); // 第三个参数，设置压缩率为不压缩（0为压缩到最小，100为最大不压缩）
    qDebug() << "done.";
```



## Qt 模型-视图-委托（Model-View-Delegate）架构

```C++
绪论：
    使用Qt模块的在QTreeView、QTableView中做自定义的窗口类型进行配置，绘制窗口。需要使用到QStyledItemDelegate这个标准的代理模块，自定义封装好需要代理的窗口。
    
使用理解：
	1.四个关键重载函数；
	QWidget *createEditor(QWidget *parent, const QStyleOptionViewItem &, const QModelIndex &index) const override;//初始化自定义类型的控件，根据QModelIndex行列判断创建的Widget的位置，(一般没有数据)。
    void setEditorData(QWidget *editor, const QModelIndex &index) const override;//利用QModelIndex中存储的数据，重新构造控件窗口，且利用QModelIndex进行参数赋值，再进行数据的修改，(!!!所修改的数据需要存储到自定义控件的成员变量中!!!)。
    void setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const override;//根据QWidget指向的指针地址通过类型转换成需要操作的自定义窗口，所得到的自定义窗口会获取到之前编辑的数据信息，通过窗口的API获取到的关键数据存储到QAbstractItemModel模型中。
    void paint(QPainter* painter, const QStyleOptionViewItem& option, const QModelIndex& index) const override;//与setEditorData函数类似，通过QModelIndex获取自定义窗口的数据，构造一个空对象，最后利用QPainter绘制这个窗口。QStyleOptionViewItem是当前位置的窗口可使用的大小及坐标位置。

	2.Qt的QMetaObject对象
    利用好Qt的元对象系统，这对自定义窗口及自定义的数据类型有很大的帮助。
        QMetaObject
        QMetaProperty
        	关键函数：//需要使用到Q_GADGET这个宏进行注册、且不能与Q_OBJECT共同使用(!!!不能使用在模板类和宏定义类中!!!)。
        	[since 5.5] QVariant QMetaProperty::readOnGadget(const void *gadget) const;
			[since 5.5] bool QMetaProperty::writeOnGadget(void *gadget, const QVariant &value) const;
	3.同时需要配合QVaraint这个动态类型，他与Qt的元对象系统密不可分。
    4.需要使用Qt的元对象系统的类成员变量还需要使用Q_PROPERTY(type registername MEMBER classMember READ readFunc WRITE writeFunc)进行注册
```

# CMake记录

```
// vs 调试信息不为乱码（命令行）
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

// 命令行窗口输出中文不乱码
#include <windows.h>
int main(){
    SetConsoleOutputCP(CP_UTF8);
}
```

