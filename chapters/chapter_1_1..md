
## SQL

### SQL 语句

我们知道网站是通过数据库来进行 CRUD 操作的，但是怎么样可以让这两者联系起来呢？SQL 就是当下用来操作数据库最流行的语言。通过以下一些简短的例子，我们可以快速的将 SQL 语句和 CRUD 操作对应起来：

![sql](../images/C1_1-sql.png)

```
# INSERT INTO => create
# SELECT => read
# UPDATE => update
# DELETE => delete
```

### 创建数据库

我们这次的 CRUD 应用是做餐馆管理应用，所以我们的数据库会包括两个东西，餐馆和菜单，大概如下：

![databse](../images/C1_1-database.png)

好，有了这个结构，我们就可以开始写代码了。

``` python
import sqlite3
conn = sqlite3.connect('restaurant.db')

c = conn.cursor()
c.execute('''
    CREATE TABLE restaurant
    (id INTEGER PRIMARY KEY ASC, name VARCHAR(250) NOT NULL)
    ''')

c.execute('''
    CREATE TABLE menu_item
    (id INTEGER PRIMARY KEY ASC, name VARCHAR(250), price VARCHAR(250),
    description VARCHAR(250) NOT NULL, restaurant_id INTEGER NOT NULL,
    FOREIGN KEY (restaurant_id) REFERENCE restaurant(id))
    ''')

conn.commit()
conn.close()
```

通过 SQL 语句我们可以完成创建两张表的工作，但是，对于 Python 来说，这只是一些语句而已。意味着当出现错别字，或者引用了一张完全不存在的表的时候， Python 并不能发挥作用。

在 Pyhon 里面，一切皆为对象，那么为什么不把数据库查询，列表都当作是对象呢？于是 ORM 就出现了！

### ORM

ORM(Object-Relational Mapping)，即对象关系映射，简单的说就是对象模型和关系模型的一种映射。

![ORM](../images/C1_1-ORM.png)

### SQLalchemy

在 python 里面，最为流行的 ORM 选择就是 [SQLalchemy](http://www.sqlalchemy.org/)，如果你用了 Vagrantfile 的话，SQLalchemy 已经安装在上面了，如果你不打算用虚拟机开发的话。请在官网上面自行下载。

#### 通过 SQLalchemy 创建数据库表格

首先我们需要创建一个 python 文件，名字叫做 database_setup.py。通常来讲，对于 SQLalchemy，创建数据库表格会有四部分组成，包括：

- 配置
    - 用于引入所需要的模块（在文件的开始）
    - 连接数据库并且导入数据（在文件的最后）
- 类名
    - 用 python 的类来表示一张表
    - 拓展 base 类
- 表
    - 指定特定的表
- 映射

##### 配置

``` python
import os
import sys
from sqlalchemy import Column, ForeignKey, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from sqlalchemy import create_engine

Base = declarative_base()

# at the end of the file
engine = create_engine('sqlite:///restaurantmenu.db')

Base.metadata.create_all(engine)
```

在上面的配置中，我们需要引进不同的模块，这里会做简要的介绍，通过 `os` 和 `sys` 模块，允许我们在不同的操作环境运行而不受干扰，同时也可以执行一些与命令行有关的交互。从 `sqlalchemy` 中我们需要引进一些常用操作数据库的方法，例如列表，整数类型，字符串类型，外键。而在  `sqlalchemy.ext.declarative` 模块中导入最基本的类，python 下的数据对象将会基于这个类扩展。通过 `sqlalchemy.orm` 我们可以引进关系类型结合外键构造两个表之间的联系。最后通过 `create_engine` 类，我们可以连接到所需要的数据库。通过 `Base` 我们可以让 SQLalchemy 知道 Base 是一个特殊的与 SQLalchemy 有映射的类。

在下面两句话中，我们分别创建了与数据库的联系和在数据库中创建相对应的表并更新信息。

##### 类名，表，映射

``` python

class Restaurant(Base):
    __tablename__ = 'restaurant'

    # mapper code here
    id = Column(Integer, primary_key=True)
    name = Column(String(250), nullable=False)

class MenuItem(Base):
    __tablename__ = 'menu_item'

    # mapper code here
    name = Column(String(80), nullable=False)
    id = Column(Integer, primary_key=True)
    description = Column(String(250))
    price = Column(String(8))
    course = Column(String(250))
    restaurant_id = Column(Integer, ForeignKey('restaurant.id'))
    restaurant = relationship(Restaurant)

```

在上面的代码中，我们继承了 Base 类并对其进行了扩展，通过 `__tablename__` 来指定我们要创建的表格，通过一些属性来进行映射。达到创建表和设置属性的目的。

好了，这个时候，我们的目录是这样的：

![dir](../images/C1_1-dir.png)

接下来只要将上面的代码放在一起，其中restaurant.db是一个空文件，到你的目录下运行`python database_setup.py`，你就成功在restaurant.db 里创建了两个表了。

## SQLalchemy 进行 CRUD 操作
