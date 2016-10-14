# Qt5.7.0-my-modified-version

#自用修改版Qt5.7.0

####这里是./qtbase/src/corelib目录下的内容。

这个修改版只修改QtCore，其他模块完全不影响。

    enum ConnectionType {
        AutoConnection,
        DirectConnection,
        QueuedConnection,
        AutoCompatConnection,
        BlockingQueuedConnection,
        ParallelBlockingQueuedConnection, //like BlockingQueuedConnection, but the receivers will begin together
        UniqueConnection =  0x80
    };
    
    enum ConnectionPosition {
        ConnectAtEnd = 0,
        ConnectAtBegin
    };
    
####ConnectionType增加了一种类型ParallelBlockingQueuedConnection，与BlockingQueuedConnection类似；但不同的是，ParallelBlockingQueuedConnection连接的槽会被同时调用，然后等到所有槽函数执行完毕后，再返回。
    void A::mySlot()
    {
        msleep(1000);
        qDebug() << "print A";
    }
    void B::mySlot()
    {
        msleep(300);
        qDebug() << "print B";
    }
    connect(sender, SIGNAL(mySignal()), A, SLOT(mySlot()), Qt::ParallelBlockingQueuedConnection);
    connect(sender, SIGNAL(mySignal()), B, SLOT(mySlot()), Qt::ParallelBlockingQueuedConnection);

    qDebug() << "begin";
    emit sender.mySignal();
    qDebug() << "end";

    输出结果为：
    begin
    (300ms)
    print B
    (700ms)
    print A
    end
    
####增加了一个枚举类型ConnectionPosition。我们知道原版Qt对于连接到同一个信号上的槽，遵循“先连接先调用”的原则，ConnectionPosition.ConnectAtBegin可以让某个槽“插队”，得到优先调用。
    connect(sender, SIGNAL(xxx), A, SLOT(xxx), Qt::AutoConnection);                         // 默认连接位置，即ConnectAtEnd
    connect(sender, SIGNAL(xxx), B, SLOT(xxx), Qt::AutoConnection, Qt::ConnectAtBegin);     // B插队到最前
    connect(sender, SIGNAL(xxx), C, SLOT(xxx), Qt::AutoConnection, Qt::ConnectAtBegin);     // C插队到最前
    这种情况下，信号触发时，ABC的调用顺序为C->B->A。
    

#使用方法
####下载源代码
Windows : http://download.qt.io/official_releases/qt/5.7/5.7.0/single/qt-everywhere-opensource-src-5.7.0.zip

Linux : http://download.qt.io/official_releases/qt/5.7/5.7.0/single/qt-everywhere-opensource-src-5.7.0.tar.gz

把这个项目的文件覆盖到qtbase/src/corelib目录，编译。

#版本兼容性
####修改版的Qt模块，仅QtCore与原版不同(多了几个导出符号)，其他模块(QtGui、QtWidgets等)理论上与原版完全相同，可以随意混用。
####修改版在代码级、二进制级向下兼容原版。
####反过来，如果代码中没有用到修改版的新特性，就可以在原版上编译（必须的）；`但是，无论是否用到新特性，使用修改版编译的程序都无法在原版上运行`。
    
    编译所用的版本           程序可以在原版上运行      程序可以在修改版上运行
    原版                    √                       √
    修改版                  ×                       √
    
    如果试图用原版dll/so上运行使用修改版编译的程序，会因为找不到专有的导出符号而出错。
    Windows会弹出错误提示框，而Linux会在第一次运行到QObject::connect时崩溃。
