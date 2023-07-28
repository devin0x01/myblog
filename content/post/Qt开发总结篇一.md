---
title: "Qt开发总结篇一"
date: 2023-06-18T14:05:25+08:00
tags: ["Qt"]
categories: []
draft: false
---
## GUI界面

### 限制输入的内容

```cpp
QValidator *validator = new QIntValidator(100, 999, this);
// 这样文本框只能输入100~999之间的数字
ui->lineEdit->setValidator(validator);
```

### 显示格式控制

```cpp
ui->textEdit->setWordWrapMode(QTextOption::WrapAnywhere); // 任意地方换行
ui->tableWidget->setTextElideMode(Qt::ElideMode); // 不显示省略号
```

## Qt序列化

通过**全局流操作运算符重载**实现，可以序列化到文件等设备中，也可以序列化到`QByteArray`中。

```cpp
// cmdinfo.h
// BaseInfo已经实现序列化
class CmdInfo : public BaseInfo {
public:
    friend QDataStream& operator>>(QDataStream&, CmdInfo&);
    friend QDataStream& operator<<(QDataStream&, const CmdInfo&);

private:
    QString command_;
    CmdType type_;
};

// cmdinfo.cpp
QDataStream& operator>>(QDataStream &in, CmdInfo &data)
{
    BaseInfo &base = data;
    qint32 tmpInt;

    in >> base >> data.command_ >> tmpInt;
    data.setType((CmdType)tmpInt);
    return in;
}
QDataStream& operator<<(QDataStream &out, const CmdInfo &data)
{
    const BaseInfo &base = data;
    out << base << data.command_ << (qint32)data.type_;
    return out;
}
```

### 序列化到文件中

```cpp
// 读
QFile file("object.dat");
file.open(QFile::ReadOnly);
QDataStream in(&file);
in >> infoObj;
file.close();

// 写
QFile file(datFilename(filename));
file.open(QFile::Truncate | QFile::WriteOnly);
QDataStream out(&file);
out << infoObj;
file.flush();
file.close();
```

### 序列化到QByteArray中

```cpp
// The QBuffer class provides a QIODevice interface for a QByteArray.
QByteArray byteArray;
QBuffer buffer(&byteArray);
buffer.open(QIODevice::WriteOnly);

QDataStream out(&buffer);
out << QApplication::palette();
```

`QDateTime`也重载了关于`QDateTime`和`QDebug`的全局流操作符

```cpp
class QDateTime {
    friend Q_CORE_EXPORT QDataStream &operator<<(QDataStream &, const QDateTime &);
    friend Q_CORE_EXPORT QDataStream &operator>>(QDataStream &, QDateTime &);

    friend Q_CORE_EXPORT QDebug operator<<(QDebug, const QDateTime &);
};
```
## QByteArray的Copy-On-Write

> QByteArray can be used to store both raw bytes (including '\0's) and traditional 8-bit '\0'-terminated strings. Using QByteArray is much more convenient than using `const char *`. Behind the scenes, it always ensures that the data is followed by a '\0' terminator, and uses [implicit sharing](https://doc.qt.io/qt-5/implicit-sharing.html) (copy-on-write) to reduce memory usage and avoid needless copying of data.

```cpp
#include <QByteArray>
#include <QDebug>

int main()
{
    char buf[] = {'1', '2', 0, '3'};
    QByteArray ba1 = QByteArray::fromRawData(buf, 4);
    QByteArray ba2 = QByteArray(buf, 4); // 深拷贝
    qDebug() << ba2.size(); // 4
    // buf的地址和ba1.constData()的地址相同
    printf("buf=%p, ba1=%p, ba2=%p\n", buf, ba1.constData(), ba2.constData());

    // 注意: data()接口会触发深拷贝，而constData不会触发深拷贝
    ba1.data();
    printf("buf=%p, ba1=%p, ba2=%p\n", buf, ba1.constData(), ba2.constData());
    return 0;
}

// 输出：
// 4
// buf=0067fea4, ba1=0067fea4, ba2=012475d8
// buf=0067fea4, ba1=0124b6f8, ba2=012475d8
```

### 底层原理

```cpp
// qbytearray.h

inline char *QByteArray::data()
{ detach(); return d->data(); }

inline const char *QByteArray::data() const
{ return d->data(); }

inline const char *QByteArray::constData() const
{ return d->data(); }

inline void QByteArray::detach()
{ if (d->ref.isShared() || (d->offset != sizeof(QByteArrayData))) reallocData(uint(d->size) + 1u, d->detachFlags()); }
```