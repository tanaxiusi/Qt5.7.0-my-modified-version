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
####增加了一个枚举类型ConnectionPosition。我们知道原版Qt对于连接到同一个信号上的槽，遵循“先连接先调用”的原则，ConnectionPosition.ConnectAtBegin可以让某个槽“插队”，得到优先调用。
    例：
    connect(sender, SIGNAL(xxx), A, SLOT(xxx), Qt::AutoConnection);                             // 默认连接位置，即ConnectAtBegin
    connect(sender, SIGNAL(xxx), B, SLOT(xxx), Qt::AutoConnection, Qt::ConnectAtBegin);         // B插队到最前
    connect(sender, SIGNAL(xxx), C, SLOT(xxx), Qt::AutoConnection, Qt::ConnectAtBegin);         // C插队到最前
    这种情况下，信号触发时，ABC的调用顺序为C->B->A
    
