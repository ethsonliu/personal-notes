https://doc.qt.io/qt-6/qobject.html#deleteLater

总结一下就是以下几点:

- 首先 deleteLater() 是 QObject 对象的一个函数, 要想使用此方法, 必须是一个 QObject 对象。
- deleteLater() 依赖于 Qt 的 event loop 机制。
- 如果在 event loop 启用前被调用, 那么 event loop 启用后对象才会被销毁;
- 如果在 event loop 结束后被调用, 那么对象不会被销毁;
- 如果在没有 event loop 的 thread 使用, 那么 thread 结束后销毁对象。
- 可以多次调用此函数。
- 线程安全。

来看看源码，

```c++
void QObject::deleteLater()
{
#ifdef QT_DEBUG
    if (qApp == this)
        qWarning("You are deferring the delete of QCoreApplication, this may not work as expected.");
#endif
    QCoreApplication::postEvent(this, new QDeferredDeleteEvent());
}

bool QObject::event(QEvent *e)
{
    switch (e->type()) {
    case QEvent::Timer:
        timerEvent((QTimerEvent *)e);
        break;

    case QEvent::ChildAdded:
    case QEvent::ChildPolished:
    case QEvent::ChildRemoved:
        childEvent((QChildEvent *)e);
        break;

    case QEvent::DeferredDelete:
        qDeleteInEventHandler(this); // 在这里
        break;
        
......
}

void qDeleteInEventHandler(QObject *o)
{
    delete o;
}

QObject::~QObject()
{
    Q_D(QObject);
    d->wasDeleted = true;
    d->blockSig = 0; // unblock signals so we always emit destroyed()

......


    if (!d->children.isEmpty())
        d->deleteChildren(); // delete 所有的孩子对象

#if QT_VERSION < 0x60000
    qt_removeObject(this);
#endif
    if (Q_UNLIKELY(qtHookData[QHooks::RemoveQObject]))
        reinterpret_cast<QHooks::RemoveQObjectCallback>(qtHookData[QHooks::RemoveQObject])(this);

    Q_TRACE(QObject_dtor, this);

    if (d->parent)        // remove it from parent object
        d->setParent_helper(nullptr); // 与父对象脱离
}
```

那什么情况下用 deletelater，什么时候直接用 delete 呢？

 If you use delete directly you must make sure that:

- There are no pending events that m_timer should receive (in this case a timerEvent) or it might crash
- You must make sure m_timer lives in the same thread as the one you are calling delete from
- You must make sure the method in which you call delete is not a slot triggerd by the object you are trying to delete or it might crash
- 
Basically, life is too short to care about all the above and the performance improvement is negligible **so using deleteLater() is just better.（大多数情况还是用 deletelater）**
