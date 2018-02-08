---
title: SQLAlchemy学习使用
date: 2018-02-08 10:30:39
tags: 数据库
categories: python
---

SQLAlchemy分为两个部分，一个是最常用的ORM对象映射，另一个是核心的SQL expression。 第一个是纯粹的ORM，后面这个不是ORM，而是DBAPI的封装，通过一些sql表达式来避免了直接写sql。 使用SQLAlchemy则可以分为三种方式。

- 使用ORM避免直接书写sql
- 使用raw sql直接书写sql
- 使用sql expression，通过SQLAlchemy的方法写sql表达式

# 安装
一般来讲对某个底层数据库需要安装相应的驱动，比如使用了mysql，那么需要安装python的mysql驱动，
有很多种选择，SQLAlchemy默认的MySQLdb/MySQL-Python，也可以使用PyMySQL,下面就用PyMySQL作为例子.
在centos上面安装MySQL-Python
1. yum install mysql-devel
2. pip install MySQL-python

安装PyMySQL: `pip install PyMySQL`

**注意：** MySQLdb仅仅支持python2，如果要支持python3，请安装PyMySQL.

# 定义映射

这里使用两个表来说明，一个用户表users，一个电子邮件表addresses，两者一对多的关系。我们先定义这两个映射：
```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

Base = declarative_base()

class Address(Base):
    """电子邮件表"""
    __tablename__ = 'addresses'

    id = Column(Integer, primary_key=True)
    email_address = Column(String(30), nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'))
    user = relationship("User", back_populates="addresses")

    def __repr__(self):
        return "<Address(email_address='{}')>".format(self.email_address)

class User(Base):
    """用户表"""
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(10))
    fullname = Column(String(20))
    password = Column(String(20))

    addresses = relationship("Address", order_by=Address.id, back_populates="user")

    def __repr__(self):
        return "<User(name='{}', fullname='{}', password='{}')>".format(
            self.name, self.fullname, self.password)
```

# 连接数据库并创建表

通过`create_engine()`可以连接数据库，另外先要提前创建test这个测试数据库：
```python
from sqlalchemy import create_engine

# 下面是MySQLdb/MySQL-Python默认写法,python2中用这个
# engine = create_engine('mysql://root:mysql@127.0.0.1:3306/test', echo=True)

# 这里我使用的是PyMySQL
# echo=True是开启调试，这样当我们执行文件的时候会提示相应的文字
engine = create_engine('mysql+pymysql://root:mysql@127.0.0.1:3306/test', echo=True)
# 定义了表映射，而数据库里面还没有真实表，这里需要使用Base类的metadata来自动创建表
Base.metadata.create_all(engine)

```

# 创建session

对数据库的操作**必须先创建一个session**，增删改查操作都有这个session负责，
首先我们先创建一个session工厂类，由它来负责后续的session创建
```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
```

# 添加用户

```python
# 先使用工程类来创建一个session对象
session = Session() 
ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
session.add(ed_user)
# 同时创建多个
session.add_all([
    User(name='wendy', fullname='Wendy Williams', password='foobar'),
    User(name='mary', fullname='Mary Contrary', password='xxg527'),
    User(name='fred', fullname='Fred Flinstone', password='blah')])
# 提交事务
session.commit()
```

# 增删查改
## 查询
在session上面调用query()方法会创建一个Query对象
```python
for user in session.query(User).order_by(User.id):
    print(user.name, user.fullname)

# 使用filter_by过滤
for name in session.query(User.name).filter_by(fullname='Ed Jones'):
    print(name)

# 使用sqlalchemy的SQL表达式语法过滤，可以使用python语句
for name in session.query(User.name).filter(User.fullname=='Ed Jones'):
    print(name)
```

## 删除
```
session.delete(ed_user)
session.query(User).filter_by(name='ed').count()

```

# 关系映射
## 一对多的关系映射
sqlalchemy使用ForeignKey来指明一对多的关系，表示一对多的关系时，在子表类中通过 foreign key (外键)引用父表类。然后，在父表类中通过 relationship() 方法来引用子表的类：比如一个用户可有多个邮件地址，而一个邮件地址只属于一个用户。那么就是典型的一对多或多对一关系。
**注意:**两个类中都通过relationship()方法指明相互关系。

在Address类中，我们定义外键，还有对应所属的user对象
```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    # 在父表类中通过 relationship() 方法来引用子表的类集合
    children = relationship("Child")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    # 在子表类中通过 foreign key (外键)引用父表的参考字段
    parent_id = Column(Integer, ForeignKey('parent.id'))
```

下面通过几个例子来操作一对多的关系映射
```python
# 先添加一个用户，并且给这个用户增加两个邮件地址
jack = User(name='jack', fullname='Jack Bean', password='gjffdd')
jack.addresses = [Address(email_address='jack@google.com'),
                  Address(email_address='j25@yahoo.com')]
session.add(jack)
session.commit()

# 查询
jack = session.query(User).filter_by(name='jack').one()
# 只有在调用jack.addresses时才会调用查询邮件地址的SQL，这个是典型的懒加载模式
jack.addresses

# join查询
session.query(User).join(Address).filter(Address.email_address=='jack@google.com').all()
```

有时候我们不想使用懒加载，而是要强制一次性加载某个关联数据，那么可以使用subqueryload或者joinedload
```python
from sqlalchemy.orm import subqueryload

jack = session.query(User).options(subqueryload(User.addresses)).filter_by(name='jack').one()

# 推荐使用下面这种方案
from sqlalchemy.orm import joinedload
jack = session.query(User).options(joinedload(User.addresses)).filter_by(name='jack').one()
```

## 多对一映射
在多对一的关系中建立双向的关系，这样的话在对方看来这就是一个多对一的关系， 在子表类中附加一个relationship()方法，并且在双方的relationship()方法中使用relationship.back_populates方法参数：
```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", back_populates="parent")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
    parent = relationship("Parent", back_populates="children")
    # 子表类中附加一个 relationship() 方法
    # 并且在(父)子表类的 relationship() 方法中使用 relationship.back_populates 参数
```
这样的话子表将会在多对一的关系中获得父表的属性,或者可以在单一的relationship()方法中使用backref参数来代替back_populates参数， 推荐使用这种方式，可以少些几句话。
```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", backref="parent")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
```

## 一对一
一对一就是多对一和一对多的一个特例,只需在relationship加上一个参数uselist=False替换多的一端就是一对一
一对多 => 一对一:
```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child = relationship("Child", uselist=False, backref="parent")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
```

多对一 => 一对一:
```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child.id'))
    child = relationship("Child", backref=backref("parent", uselist=False))

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
```

## 多对多
多对多关系需要一个中间关联表,通过参数secondary来指定。backref会自动的为子表类加载同样的secondary参数, 
所以为了简洁起见仍然推荐这种写法：
```python
from sqlalchemy import Table, Text

post_keywords = Table('post_keywords',Base.metadata,
    Column('post_id',Integer,ForeignKey('posts.id')),
    Column('keyword_id',Integer,ForeignKey('keywords.id'))
)

class BlogPost(Base):
    __tablename__ = 'posts'
    id = Column(Integer,primary_key=True)
    body = Column(Text)
    keywords = relationship('Keyword',secondary=post_keywords,backref='posts')

class Keyword(Base):
    __tablename__ = 'keywords'
    id = Column(Integer,primary_key = True)
    keyword = Column(String(50),nullable=False,unique=True)
```

如果使用back_populates，那么两个都要定义：
```python
from sqlalchemy import Table, Text
post_keywords = Table('post_keywords',Base.metadata,
    Column('post_id',Integer,ForeignKey('posts.id')),
    Column('keyword_id',Integer,ForeignKey('keywords.id'))
)

class BlogPost(Base):
    __tablename__ = 'posts'
    id = Column(Integer,primary_key=True)
    body = Column(Text)
    keywords = relationship('Keyword',secondary=post_keywords,back_populates="parents")

class Keyword(Base):
    __tablename__ = 'keywords'
    id = Column(Integer,primary_key = True)
    keyword = Column(String(50),nullable=False,unique=True)
    parents = relationship('BlogPost',secondary=post_keywords,back_populates="keywords")
```
# 一些重要参数
relationship()函数接收的参数非常多，比如：backref，secondary，primaryjoin等等。 下面列举一下我用到的参数

- backref 在一对多或多对一之间建立双向关系
- lazy:默认值是True, 懒加载
- remote_side: 表中的外键引用的是自身时, 如Node类,如果想表示多对一的树形关系, 那么就可以使用remote_side
```python
class Node(Base):
__tablename__ = 'node'
id = Column(Integer, primary_key=True)
parent_id = Column(Integer, ForeignKey('node.id'))
data = Column(String(50))
parent = relationship("Node", remote_side=[id])
```
- secondary: 多对多指定中间表关键字
- order_by: 在一对多的关系中,如下代码:
```python
class User(Base):

  addresses = relationship(lambda: Address,
                   order_by=lambda: desc(Address.email),
                   primaryjoin=lambda: Address.user_id==User.id)
```
- cascade: 级联删除
```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer,primary_key = True)
    children = relationship("Child",cascade='all',backref='parent')

def delete_parent():
    session = Session()
    parent = session.query(Parent).get(2)
    session.delete(parent)
    session.commit()
```
不设置cascade，删除parent时，其关联的chilren不会删除，只会把chilren关联的parent.id置为空， 
设置cascade后就可以级联删除children

# 对象的四种状态
对象在session中可能存在的四种状态包括：

- Transient：实例还不在session中，还没有保存到数据库中去，没有数据库身份，像刚创建出来的对象比如User()，仅仅只有mapper()与之关联
- Pending：用add()一个transient对象后，就变成了一个pending对象，这时候仍然没有flushed到数据库中去，直到flush发生。
- Persistent：实例出现在session中而且在数据库中也有记录了，通常是通过flush一个pending实例变成Persistent或者从数据库中querying一个已经存在的实例。
- Detached：一个对象它有记录在数据库中，但是不在任何session中，

# 关联查询
查询：[http://docs.sqlalchemy.org/en/rel_1_1/orm/query.html]
关联查询: [http://docs.sqlalchemy.org/en/rel_1_1/orm/query.html#sqlalchemy.orm.query.Query.join]

```python
# 非常简单的关联查询，外键就一个，系统知道如何去关联：
session.query(User).join(Address).filter(Address.email==”lzjun@qq.com”).all()
# 指定ON字段：
q = session.query(User).join(Address, User.id==Address.user_id)
# 多个join
q = session.query(User).join("orders", "items", "keywords")
q = session.query(User).join(User.orders).join(Order.items).join(Item.keywords)
# 子查询JOIN：
address_subq = session.query(Address).\
                filter(Address.email_address == 'ed@foo.com').\
                subquery()

q = session.query(User).join(address_subq, User.addresses)
# join from:
q = session.query(Address).select_from(User).\
                join(User.addresses).\
                filter(User.name == 'ed')
# 和下面的SQL等价：
SELECT address.* FROM user
    JOIN address ON user.id=address.user_id
    WHERE user.name = :name_1
# 左外连接，指定isouter=True，等价于Query.outerjoin()：
q = session.query(Node).\
        join("children", "children", aliased=True, isouter=True).\
        filter(Node.name == 'grandchild 1')
```
# 参考
* [SQLAlchemy入门](https://www.xncoding.com/2016/03/07/python/sqlalchemy01.html)