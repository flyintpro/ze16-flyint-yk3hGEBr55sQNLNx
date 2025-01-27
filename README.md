
### 02 简单的增删查改


前情提要：承接了01中的`engine`以及`User`类


#### 2\-1 了解会话机制


1. 个人理解


在SQLAlchemy 增删查改中是依赖会话（Session）这个机制进行操作的，我个人的理解是用“会话“进行连接数据库周期的一系列管理操作（以下是ai生成对此会话的理解）
2. ai理解


在 SQLAlchemy 中，**`sessionmaker`** 是用于创建 **会话（Session）** 对象的工厂类，负责与数据库的交互。会话（Session）是 SQLAlchemy ORM 的核心部分，它管理与数据库的连接，并在一次操作中追踪对象的状态变化（如添加、修改、删除），同时处理事务的提交和回滚。


**会话（Session）的作用**


	1. **管理事务**：会话负责开始、提交和回滚事务。通过会话，SQLAlchemy 可以在多个数据库操作间提供原子性（即要么全部成功，要么全部失败）。
	2. **对象的持久化**：会话在内存中追踪对象的状态变化，确保对象在数据库中得到正确的插入、更新或删除。
	3. **查询管理**：会话管理 SQL 查询的生命周期，包括从数据库获取数据，执行查询语句等。


#### 2\-2 创建会话


* 流程：


	1. 导入对应包，依赖其中的 `sessionmaker`
	2. 使用[`sessionmaker`](https://github.com) 生成新的会话对象 [`Session`](https://github.com) ，生成时绑定之前创建过的引擎
	3. 通过`Session`对象我们来构造对应的实例【可能有点绕，但一开始2调用的只是我们绑定好的类，我们还需要对这个对象创造实例来进行会话管理操作】
* 官方的用例参考，他用的数据库类型不一样，但是思路是一样的：


**eg1：**



```


|  | from sqlalchemy import create_engine |
| --- | --- |
|  | from sqlalchemy.orm import sessionmaker |
|  |  |
|  | # an Engine, which the Session will use for connection |
|  | # resources |
|  | engine = create_engine("postgresql+psycopg2://scott:tiger@localhost/") |
|  |  |
|  | Session = sessionmaker(engine) |
|  |  |
|  | with Session() as session: |
|  | session.add(some_object) |
|  | session.add(some_other_object) |
|  | session.commit() |


```

**eg2**：这个就是后文使用的创建实例进行操作了



```


|  | session = Session() |
| --- | --- |
|  | try: |
|  | session.add(some_object) |
|  | session.add(some_other_object) |
|  | session.commit() |
|  | finally: |
|  | session.close() |


```

**eg3**：



```


|  | Session = sessionmaker(engine) |
| --- | --- |
|  |  |
|  | with Session.begin() as session: |
|  | session.add(some_object) |
|  | session.add(some_other_object) |
|  | # commits transaction, closes session |


```
* 看完官方的实例，来写我们自己的会话管理了：


	1. 简化版：
```


|  | from test_package_sql import engine, User #这个只是导入01文件提及的engine和User类，名字自己改一下 |
| --- | --- |
|  | from sqlalchemy.orm import sessionmaker |
|  |  |
|  | Session = sessionmaker(bind=engine)#先通过时间创造器绑定引擎->创建这个会话类 |
|  |  |
|  | session = Session()#根据我们自己的Session类创建属于我们自己的实例对象用于后续操作 |


```

	2. 详细版：
	
	
	
	```
	
	
	|  | from sqlalchemy import create_engine, Column, Integer, Float, String |
	| --- | --- |
	|  | from sqlalchemy.orm import declarative_base |
	|  |  |
	|  | from sqlalchemy.orm import sessionmaker |
	|  |  |
	|  | local_url = "sqlite:///02_test.db" |
	|  | engine = create_engine(url=local_url) |
	|  | local_base = declarative_base() |
	|  | class User(local_base): |
	|  | __tablename__ = "User" |
	|  | id = Column(Integer, primary_key=True) |
	|  | name = Column(String) |
	|  | age = Column(Integer) |
	|  |  |
	|  | local_base.metadata.create_all(engine) |
	
	
	```


#### 2\-3 增


1. 事前bb【觉得烦的可以略过】


在进行增删查改前，必须再强调一下这个会话机制的流程：**创建会话\-\>进行管理操作（包括但不不限于增删查改）\-\>提交会话管理**【注意，提交时的更改的只是当前会话状态，言下之意，你提交了一次再修改，没提交那还是之前提交的状态】
2. 流程介绍【增加单个**User对象**】


	* 创建需要增添的实例——**必须与User类相匹配**，就是说是User类对象
	* 使用对话机制的 `session.add`接口进行添加
	* 添加若干个后，结束则使用 `session.commit()` 进行修改后的提交
	
	
	*主函数直接调用就好，我这样子写好封装，看得也清楚，很简单对吧*
```


|  | def add_one(): |
| --- | --- |
|  | User1 = User(name="arthur",age=18) |
|  | User2 = User(name="lihua",age=21) |
|  | session.add(User1) |
|  | session.add(User2) |
|  | session.commit() |


```

	* 增添效果如下【db为空时】：
	
	
	![image-20250108000748412](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154326499-1441703222.png)
	
	
	下一集我再对如何显示数据库做一个解释吧，毕竟是从零开始（bushi）
3. 增添多个对象【或者是User\_list】


只需要修改接口以及传入参数就好了，使用 `session.add_all()`接口



```


|  | def add_Users(): |
| --- | --- |
|  | User1 = User(name="arthur", age=18) |
|  | User2 = User(name="lihua", age=21) |
|  | Users = [User1,User2] |
|  | session.add_all(Users) |
|  | session.commit() |


```

	* explain：Users就是User 类（项目）的可迭代对象（列表），让我们看看官方对于这个接口的解释：
> *method* [`sqlalchemy.orm.Session.`](https://github.com)**add\_all**(*instances: Iterable\[object]*) → None
> 
> 
> Add the given collection of instances to this [`Session`](https://github.com):[豆荚加速器官网PodHub](https://doujiaa.com).
> 
> 
> 添加实例到Seesion会话中。这句话可能不太好理解，看看他的传参解释其实就很直观，其实就是传入可迭代对象XD


	* output：
	
	
	![image-20250108001929442](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154327499-477487735.png)
	
	
	*就结束了，其实增删改查不难，接口已经抽象出来了，记住逻辑就好*


#### 2\-4 查


还是得先说查啊，不然没法删啊0v0


1. 接口介绍


`q = session.query(SomeMappedClass)`


	* 参数介绍：
	
	
	`SomeMappedClass`：所映射的类
	
	
	`q`：返回的类型是`Query[Any]`，即任何满足要求的数据
	* 常用后缀
	
	
		1. `.all()`：以列表的形式传回搜索的结果
	```
	
	
	|  | session.query(User).all()#用来返回User表的所有数据 |
	| --- | --- |
	
	
	```
	
		2. [`sqlalchemy.orm.Query.`](https://github.com)**filter\_by**(\* \*kwargs)[¶](https://github.com) 这个是常用的简化版的搜索接口。**！注意他可以同时接收多个变量,且只使用等值过滤条件**
		
		
		
		```
		
		
		|  | results = session.query(User).filter_by(ag,e=21) |
		| --- | --- |
		|  | results = session.query(User).filter_by(age=21，name="lihua") |
		
		
		```
		3. `session.query(Model).filter(condition)`
		
		
		运行使用多个复杂的表达式，支持更多条件类型，**支持手动连接多个条件**：如果需要多个条件，可以使用 `AND` 或 `OR` 手动连接条件。但就说注意对比时需要`User.xxx`\-\-你自己定义的类表
		
		
		
		```
		
		
		|  | results = session.query(User).filter(User.name == 'arthur')#不止支持== ，还支持其他比较运算符 |
		| --- | --- |
		
		
		```
		4. `.one_or_none()`
		
		
		用于查找**是否只有一个**或者**none个**，找到多个则会报错
	2. 编码
	
	
	其实把上面的看懂了就已经学会了如何搜索了
	
	
		* 简单搜索法：
		
		
		
		```
		
		
		|  | def search(): |
		| --- | --- |
		|  | results = session.query(User).filter_by(age=21) |
		|  | for result_one in results: |
		|  | print(result_one) |
		
		
		```
		
		![image-20250108150708363](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154326066-39507940.png)
		* 使用传统接口更多扩展的的搜索：
		
		
		
		```
		
		
		|  | def other_way_search(): |
		| --- | --- |
		|  | results = session.query(User).filter(User.name == 'arthur') |
		|  | for result_one in results: |
		|  | print(result_one) |
		
		
		```


#### 2\-5 删


* 删除主要运用到 `session.delete(object)`接口，主要有以下两种用法：


1. 删除单个对象：



```


|  | session.delete(object)#object是单个对象 |
| --- | --- |
|  | session.commit() |


```

2. 删除多个满足条件的对象



```


|  | session.query(Model).filter(condition).delete() |
| --- | --- |
|  | session.commit() |


```

流程大概是：**找到想要删除的对象\-\>删除\-\>提交**


* code：



```


|  | def remove_one():#单个 |
| --- | --- |
|  | results = session.query(User).filter_by(age=21).first()#找到第一个 |
|  | session.delete(results) |
|  | session.commit() |


```

原数据库：


![image-20250108151915387](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154325974-1775570302.png)


删除后：


![image-20250108151952755](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154326616-498309847.png)



```


|  | def remove_all():#多个 |
| --- | --- |
|  | results = session.query(User).filter_by(name='arthur').delete() |
|  | session.commit() |


```

删除后：


![image-20250108152247865](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154339159-1683584448.png)
* 删除尾声：


删除就结束了，最后各位可以尝试把lihua也删除，然后根据上面的教程添加再添加4个User信息，其中要有两个age、name不一样


参考code：



```


|  | def add_4new_user(): |
| --- | --- |
|  | User1 = User(name='arthur', age=18) |
|  | User2 = User(name='Abigail Williams',age=14) |
|  | User3 = User(name='caster', age=21) |
|  | User4 = User(name='Lilith', age=22) |
|  | users = [User1,User2,User3,User4] |
|  | session.add_all(users) |
|  | session.commit() |


```

结果：


![image-20250108153103874](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154331375-256322814.png)


#### 2\-6 改


改其实最简单，找到对应的`object`然后修改，提交即可，非常朴实无华



```


|  | def update_f(): |
| --- | --- |
|  | caster = session.query(User).filter_by(name="caster").one_or_none()#找到一个caster的信息 |
|  | caster.age = 18			#修改就好啦 |
|  | session.commit()		#记得提交 |


```

![image-20250108153851400](https://img2024.cnblogs.com/blog/3244923/202501/3244923-20250108154326152-861911700.png)


part2结束，感谢观看，读者可以自行实践一下增删查改，最好融合起来一起实践


