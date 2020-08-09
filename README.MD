# Hibernate

## 第一节 什么是Hibernate

    ·一个框架
    ·一个Java领域的<持久化>框架
    ·一个<ORM>框架

    1.1 对象的持久化

        狭义的理解：

            “持久化”指把对象永久保存到数据库中

        广义的理解：

            “持久化”包括和数据库相关的各种操作：
             - 保存
             - 更新
             - 删除
             - 查询
             - 加载：根据特定的OID，把一个对象从数据库加载到内存

                > OID: 
                    为了在系统中能够找到所需对象，需要为每一个对象分配一个唯一的标识号。
                    在关系型数据库中称之为主键，而在对象术语中，
                    称作对象标识(Object Identifier - OID)

    1.2 ORM - Object/Relation Mapping: 对象/关系映射

        ·ORM主要解决对象-关系的映射

|面向对象|面向关系|
|---|---|
|类|表|
|对象|表的行(记录)|
|属性|表的列(字段)|

        ·ORM的思想：
            将关系数据库表中的记录映射成为对象，以对象的形式展现，
            程序员可以把对数据库的操作转化为对对象的操作。

        ·ORM采用<元数据>来描述对象-关系映射细节，元数据通常采用XML格式，
         并且存放在专门的<对象-关系映射文件>中。

    1.3 ORM架构图

    业务逻辑层      域模型                      <-------
                （对象、属性、关联、继承、多态）         |
                                                      |
    持久化层        ORM API ☆                   对象-关系映射文件 ☆
                   ORM实现    ------- 参照 ---->    （XML）
                                                      |
    数据库层        关系型数据模型                      |
                （表、字段、索引、主键、外键）    <------|

    1.4 流行的ORM框架

        ·Hibernate

            - 非常优秀、成熟的ORM框架
            - 完成对象的持久化操作
            - Hibernate允许开发者采用面向对象的方式来操作关系数据库
            - 消除针对特定数据库厂商的SQL代码

        ·MyBatis

            - 比Hibernate灵活度高，运行速度快
            - 开发速度慢，不支持纯粹的面向对象操作，需要熟悉SQL语句，

              并且熟练使用SQL语句优化功能

        

        ·TopLink

        

        ·OJB

## 第二节 Hello Hibernate

    ·创建Hibernate配置文件
    ·创建持久化类
    ·创建对象-关系映射文件
    ·通过HibernateAPI编写访问数据库的代码

    2.1 创建Hibernate配置文件

    <session-factory>
    	<!-- 1 配置连接数据库的基本信息 -->
		<property name="connection.username">root</property>
		<property name="connection.password">root</property>
		<property name="connection.driver_class">com.mysql.jdbc.Driver</property>
		<property name="connection.url">jdbc:mysql://localhost:3306/hibernate5</property>
		<!-- <property name="connection.url">jdbc:mysql:///hibernate5</property> -->

		<!-- 2 配置Hibernate的基本信息 -->
		<!-- 2.1 Hibernate所使用的数据库方言 -->
		<property name="dialect">org.hibernate.dialect.MySQL8Dialect</property>
		<!-- 2.2 执行操作时是否在控制台打印SQL -->
		<property name="show_sql">true</property>
		<!-- 2.3 是否对SQL进行格式化 -->
		<property name="format_sql">true</property>
        <!-- 2.4 指定自动生成数据表的策略 -->
		<property name="hbm2ddl.auto">update</property>
    </session-factory>

        ·Hinbernate从其配置文件中读取和数据库连接的有关信息，
         这个文件应该位于应用的classpath下。

    2.2 创建持久化对象

        ·pt.hibernate.News

            - Integer id
            - String title
            - String author
            - Date date

        ·提供一个无参构造器
        ·提供一个标识属性，通常映射为主键。如果没有该属性，一些功能将不起作用。
        ·为类的持久化字段声明访问方法（Getter/Setter）
        ·使用非final类

            - 在运行时生成代理是Hibernate的一个重要功能。

              如果持久化类没有实现任何接口，Hibernate使用CGLIB生成代理。
              如果类属性为final，则无法生成CGLIB代理。
        ·重写equals和hashCode方法

        ※Hibernate不要求持久化类继承任何父类或接口，因此被称为低侵入式设计

    2.3 根据持久化对象自动生成映射文件，并向Hibernate配置文件中添加它

        ! Something went wrong here, and Eclipse won't work
        ! Though DB to mapper works well
        ! Handy write a mapper

        - NewsEntity.hbm.xml

            <hibernate-mapping>
                <class name="pt.joja.warehouse.entities.NewsEntity" table="NEWS" schema="hibernate5">
                    <id name="id" type="java.lang.Integer">
                        <column name="ID"/>
                        <!-- 指定主键的生成方式 native：数据库本地方式 -->
                        <generator class="native"/>
                    </id>
                    <property name="title" type="java.lang.String">
                        <column name="TITLE"/>
                    </property>
                    <property name="author" type="java.lang.String">
                        <column name="AUTHOR"/>
                    </property>
                    <property name="date" type="java.util.Date" >
                        <column name="DATE"/>
                    </property>
                </class>
            </hibernate-mapping>

        - hibernate.cfg.xml

            <mapping resource="pt/joja/warehouse/entities/NewsEntity.hbm.xml"/>

            ※指定目录路径，不是包名

    2.4 通过HibernateAPI编写访问数据库的代码

        ·创建SessionFactory对象
        ·从SessionFactory获取Session
        ·从Session开启事务
        ·向Session对象发行数据库的操作
        ·提交/回滚事务
        ·关闭Session
        ·关闭SessionFactory

## 第三节 Hibernate常用API

    3.1 Configuration类

        ·Configuration类负责管理Hibernate的配置信息，包括以下内容：

            - Hibernate运行的底层信息：

                数据库URL
                用户名
                密码
                JDBC驱动类
                数据库方言
                连接池
                ...

            - 持久化类数据库表的映射关系（*.hbm.xml文件）

    
        ·创建Configuration的两种方式：

            - properties文件

                Configuration configuration = new Configuration();

            - xml文件

                Configuration configuration = new Configuration().configure();

                ※重载方法有configure(String path)/configure(File cfgFile)

        ※Hibernate配置文件的一个选项：
            ·hbm2ddl.auto 
                该选项可以帮助程序员实现正向工程，即由java生成数据库脚本，进而生成表结构。
                可选值：

                    - create

                        根据.hbm.xml生成数据表，每次都将表重置

                    - create-drop

                        根据.hbm.xml生成数据表，但是SessionFactory一旦关闭表就自动删除

                    - upate

                        最常用的值。
                        根据.hbm.xml生成数据表，若.hbm.xml的映射结构和现有表不同，
                        则更新数据表且保留既有数据

                    - validate

                        检查.hbm.xml和数据表的结构，不一致时抛出异常

    3.2 SessionFactory接口

        ·针对单个数据库映射关系经过编译后的内存镜像，是线程安全的
        ·SessionFactory对象一旦构造完毕，即被赋予特定的配置信息
        ·SessionFactory是生成Session的工厂
        ·构造SessionFactory很消耗资源，通常情况下一个项目中只初始化一个SessionFactory对象
        ·所有基于Hibernate的配置或服务都必须统一向ServiceRegistry注册后才能生效

    3.3 Session接口

        JDBC

        Database
            |
        JDBC Connection(s)
            |   |
            |   ConnectionPool
            |   |
        Client Application

        Hibernate

        Database
            |
        JDBC Connection
            |
        Sessions
            |   |
            |   SessionFactory
            |   |
        Client Application

        ·Session是应用程序与数据库交互操作的一个单线程对象。
         Hibernate对持久化对象的所有操作都在Session管理下进行。
        ·Session对象的生命周期很短
        ·Session对象有一个一级缓存，
         显式执行flush之前，所有的持久层操作的数据都缓存在session对象处。

        ·常用方法：

            - get()/load()
            - save(), update(), saveOrUpdate(), delete()
            - beginTrasaction()
            - isOpen(), flush(), clear(), evict(), close()

            ...
    
    3.4 Trasaction(事务)

        ·代表一次原子性操作，对应数据库的事务的概念。
         所有持久化层都应该在事务管理下进行，即使是只读操作。

        ·常用方法：

            - commit()
            - rollback()
            - wasCommitted()

## 第四节 通过Session操作对象

    4.1 Session概述

        ·Session提供了基本的保存，更新，删除和加载Java对象的方法

        ·Session有一个缓存，位于缓存中的对象成为Java持久化对象，和数据库的相关记录对应
         
        ·站在持久化角度，Hibernate把对象分成4种状态：

            - 持久化状态
            - 临时状态
            - 游离状态
            - 删除状态

         Session的特定方法使对象从一个状态转换到另一个状态

    4.2 Session缓存

        ·在Session接口的实现中包含一系列的Java集合，这些Java集合构成了Session的缓存。
         只要Session实例没有结束生命周期，而且没有清理缓存，
         那么存放在它缓存中的对象也不会结束生命周期。
        ·Hibernate缓存可以减少应用程序访问数据库的频率。

        ·操作Session缓存：

            - flush()

                将缓存中对象同步到数据库

            - reflesh()

                将数据库中记录同步到缓存

            - clear()

                清空缓存

        ·设定刷新缓存的时间点

            - setFlushMode()

| 清理缓存的模式         | 各种查询方法 | Transaction.commit() | Session.flush() |
|----------------------|------------|----------------------|-----------------|
| FlushMode. ATUO(默认) | 清理       | 清理                 | 清理            |
| FlushMode. COMMIT    | 不清理      | 清理                 | 清理            |
| FlushMode. NEVER     | 不清理      | 不清理                | 清理            |

        ·在Hibernate配置文件中设置DB连接的事务隔离级别

            <properties name="hibernate.connection.isolation">1</properties>

                - 1 ：READ UNCOMMITTED
                - 2 ：READ COMMITTED
                - 4 : REPEATABLE READ
                - 8 ：SERIALIZABLE

    4.3 持久化对象的状态转换

        ·临时对象（Transient）

            - 在使用代理主键的情况下，OID通常为null
            - 不处在Session的缓存中
            - 在数据库中没有对应的记录

        

        ·持久化对象/托管（Persist）

            - OID不为null
            - 位于Session缓存中
            - 和数据库中相关的记录对应
            - Session在flush缓存时，会根据持久化对象的属性变化来同步更新数据库
            - 在同一个Session实例的缓存中，数据库表中的每条记录只对应唯一的持久化对象

        ·删除对象（Removed）

            - 在数据库中没有和其OID对应的记录
            - 不再处于Session缓存中
            - 一般情况下，应用程序不应该继续使用该对象

        

        ·游离对象/脱管（Detached）

            - OID不为null
            - 不再处于Session缓存中
            - 一般情况下，游离对象是从持久化对象转变来的，因此数据库中可能还存在对应的记录

        Object ----- new -----> 临时状态 --------------------------------------
          |                     |                                             |
        get()                   save()                                        |
        load()                  saveOrUpdate()                                |
        Query.list()            persist()                                     |
        Query.uniqueResult()    merge()                                       |
        Query.iterator()        |                                             |
        Query.scroll()          |                                             |
          |                     v                                             v
          -------------------> 持久化状态 ------ delete() -----> 删除状态 -- 垃圾回收
                                |   ^                               ^         ^
                                |   |                               |         |
                            evict() update()                       delete()   |
                            close() saveOrUpdate()                  |         |
                            clear() merge()                         |         |
                                |   |                               |         |
                                v   |                               |         |
                                游离状态 ---------------------------------------

    4.4 save()和persist()方法

        ·save()

            1. 使一个临时对象变为持久化对象
            2. 为对象分配ID
            3. 在flush缓存时会发送一条insert语句
            4. 在save方法之前设置id是无效的
            5. 持久化对象的id是不能被修改的

            

        ·persist()

            1. persist()基本和save()方法一样
            2. 在persist()方法调用之前若对象已经有id，则不会执行insert，而会抛出异常

    4.5 get()和load()

        ·get() vs load()

            1. get()方法会立即加载对象，

               load()方法若不使用该对象，则不会立即执行查询操作，而返回一个代理对象
               - get()是立即检索，load()是延迟检索

            2. load()可能会抛LazyInitializationException

                （在需要取得代理对象时已经关闭了session）

            3. 若表中没有对应记录，get()返回null

                在Session没有关闭的前提下使用对象时，load()抛出异常

    4.6 update()

        1. 若更新一个持久化对象，不需要显式调用update()方法

            因为在调用Transaction.commit()时会先执行session.flush()

        2. 若要更新一个游离对象，需要显式调用session.update()方法

            同时会把游离对象变成持久化对象

        3. 游离对象调用update()时，无论对象和数据表的记录是否一致，都会发送SQL语句

            - 避免update()盲目触发SQL:

                在.hbm.xml的class节点中设置select-before-update=true即可
                ※但是会使正常执行更新时发送额外的SQL

        4. 若数据库中没有对应记录而调用了update()方法，则会抛出异常

        5. 游离对象调用update()时，若session缓存中已经有了OID相同的持久化对象则抛出异常

    4.7 saveOrUpdate()

        1. 以对象的OID是否为null来判断执行哪个方法

                null -> save()
                not null -> update()

        2. 若OID不为空，则会抛出异常

        3. OID值等于id的unsaved-value属性值对象，也被认为是一个临时对象

    4.8 merge()

        -> 在JPA里详细解说

    4.9 delete()

        1. 只要OID不为空就可以进行删除 ※游离对象也OK

        2. 可以通过配置.cfg.xml的属性，使delete()过后对象UID置空

            hibernate.use_identifier_rollback -> true

    4.10 evict()

        从session缓存中删除某个持久化对象

    4.11 doWork()

        实装org.hibernate.jdbc.Work接口来获取jdbc driver的原装Connetion对象

## 第五节 Hibernate配置文件 追加配置1

    5.1 C3P0数据库连接池

        步骤：

            1. 导入jar包

                hibernate安装包\optional\c3p0\*.jar

            2. 加入配置
                - hibernate.c3p0.max_size 连接池最大连接数
                - hibernate.c3p0.min_size 连接池最小连接数
                - hibernate.c3p0.acquire_increment 连接耗尽时，一次性获取多少新连接
                - hibernate.c3p0.timeout 连接池中连接多久未使用后被销毁
                - hibernate.c3p0.idle_test_period 多久检测一次池内连接是否超时
                - hibernate.c3p0.max_statements 缓存Statement对象的数量

    5.2 一次性读取的记录条数

        hibernate.jdbc.fetch_size

        100

    5.3 批量操作时的批次大小

        hibernate.jdbc.batch_size

        30
          

## 第六节 对象关系映射文件

    6.1 映射文件说明

        - 类层次 class

            ·主键 id
            ·基本类型 property
            ·实体引用类 many-to-one | one-to-one
            ·集合 set | list | map | array

                - one-to-many
                - many-to-many

            ·子类 subclass | joined-subclass
            ·其他 component | any ...

        - 查询语句 query

    6.2 hibernate-mapping元素

        - package 可选包名
        - schema
        - catalog
        - default-cascade
        - default-access
        - default-lazy
        - auto-import

    6.3 class元素

        - name 类名
        - table 表名，默认值是类名
        - dynamic-insert 发行的SQL仅包含值不为null的项目
        - dynamic-update 发行的SQL仅包含值变更的项目
        - select-before-update
        - batch-size
        - lazy
        - mutable
        - discriminator-value

    6.4 id元素

        - name OID属性名
        - type Hibernate映射类型
        - column 列名（主键字段名）
        - generator
            - class 标识符生成器类名（缩写名也可）
        - unsaved-value 指定临时对象OID

    6.5 各种generator

        - increment 

            Hibernate以递增方式为代理主键赋值，存在并发问题故不宜用于生产环境
            ※OID必须为long, int, short

        - identity 

            由数据库来自动生成
            ※要求底层数据库把主键字段设为自增，且OID必须为long, int, short

        - sequence

            由数据库生成的序列来生成
            ※要求底层数据库支持序列，且OID必须为long, int, short

        - hilo

            由Hibernate按照高低算法生成标识符（会自动在数据库中建一个用于生成主键的表）
            ※不依赖与底层数据库，但是OID必须为long, int, short

        - seqhilo
        - native

            根据底层数据库的支持能力自动选择identity，sequence或hilo来生成标识符
            ※OID必须为long, int, short

        - uuid.hex
        - uuid.string
        - assigned
        - foreign

    6.6 property元素

        - name 属性名
        - type 映射类型

            ·指定java类型，则根据java类型选择对应的Hibernate类型
            ·指定Hibernate类型，则采用

        - column 列名
        - not-null 非空约束
        - access 访问策略，getter+setter or 反射访问属性
        - unique 唯一约束
        - index 创建索引
        - length 长度
        - scale 小数位数
        - formula SQL表达式，用于计算派生属性的值

    // Maybe someday later...