Sqlalchemy 学习笔记
==============================================================================

.. contents::
    :local:


Sqlalchemy 介绍
------------------------------------------------------------------------------

SQLAlchemy 的中文简介: "一个知名企业级的持久化模式的，专为高效率和高性能的数据库访问设计的，改编成一个简单的Python域语言的完整套件"。其功能极其完整和强大, 学习曲线对新手可能不够平滑, 但绝对是最值得长期投入学习的库。

此项目是对SQLAlchemy的一些学习笔记和使用案例代码。

其他值得学习的ORM框架:

- peewee: http://docs.peewee-orm.com/en/latest/index.html
    - 优点:
        - Django式的API, 使其易用。
        - 轻量实现, 很容易和任意web框架集成。
    - 缺点:
        - 不支持自动化 schema 迁移。
        - 多对多查询写起来不直观。

- django ORM: django框架自带的ORM, https://docs.djangoproject.com/en/1.9/topics/db/models/
    - 优点:
        - 易用, 学习曲线短。
        - 和Django紧密集合, 用Django时使用约定俗成的方法去操作数据库。
    - 缺点:
        - 不好处理复杂的查询, 强制开发者回到原生SQL。
        - 紧密和Django集成, 使得在Django环境外很难使用。
- ponyorm: https://ponyorm.org/
    - 一个神奇的库, 通过解析字节码, 将生成器推导语法转化成查询, 非常牛逼的库.


测试数据库环境
------------------------------------------------------------------------------

推荐使用两个数据库, sqlite 和 postgresql.

- sqlite 可以直接用 ``engine = sqlalchemy.create_engine(":memory:")`` 建立内存数据库
- 在 Mac 系统上, postgresql 可以直接用 ``bash ./bin/run-postgres.sh`` 使用 docker 启动一个数据库, 连接信息请参考 ``./bin/run-postgres.sh`` 文件. 用 ``bash ./bin/stop-postgres.sh`` 可以杀死数据库进程以及数据. 在 Mac 上安装 docker 非常简单, 到主页下载 dmg 安装即可.


engine, connection, session
------------------------------------------------------------------------------

- engine 只是定义了引擎的一个内存数据结构, 比如: dbapi 版本, 以及对应的 Python 库. host, port, database, username, password 等等, **engine 本身不会真正地跟数据库建立连接**
- connection 是逻辑上跟数据库的连接, 同一时间能同时保持的数据库的连接是有限的. 和数据库建立真正的连接非常消耗资源, 需要身份验证, 多次握手确认才能达成.
- session 是 ORM 框架中的上下文环境, 在一个 session 中的任何数据修改都会被记录在内存中. session 是更加轻量级的对 connection 的再利用.

Sqlalchemy 的解决方案是, 维护一个连接池 (pool), 假设能容纳 200 个连接. 当执行: ``connect = engine.connect()`` 或是 ``Session = sessionmaker(bind=engine); session = Session()`` 时, 就会创建一个 **真实的数据库连接**. 而当 ``connect.close()`` 或是 ``session.close()`` 时. 在本地的内存里, 连接已经被释放了, 但是在数据库那边, 还认为我们依然连接着. 而这个在本地被 "释放" 的连接, 则被放回了连接池中, 随时准备再次使用. 而再次被使用时, ``engine.connect()``, ``Session()`` 并没有真正创建连接, 而是逻辑上 "创建" 了连接.

- ``engine = create_engine()`` 在全局只定义一次, 然后在其他地方 import.
- ``Session = sessionmaker(bind=engine)`` 只定义一次, 然后在其他地方临时创建 ``session = Session()``.

Reference:

- Engine: https://docs.sqlalchemy.org/en/latest/core/engines.html
- Connection Pool: https://docs.sqlalchemy.org/en/latest/core/pooling.html
- Session: https://docs.sqlalchemy.org/en/latest/orm/session.html


一些有用的 API
------------------------------------------------------------------------------

有一些 API 非常有用, 大多来自于这篇文档 https://docs.sqlalchemy.org/en/latest/orm/query.html
