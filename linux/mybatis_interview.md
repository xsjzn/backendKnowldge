# MyBatis面试题

## 1、#{}和${}的区别是什么？

- `${}`是 Properties 文件中的**变量占位符**，它可以用于**标签属性值和 sql 内部，属于静态文本替换**，比如${driver}会被静态替换为`com.mysql.jdbc.Driver`。



- `#{}`是 **sql 的参数占位符**，**会进行预编译处理**，MyBatis 会将 sql 中的`#{}`替换为?号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的?号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的**取值方式为使用反射从参数对象中获取 item 对象的 name 属性值**，相当于 `param.getItem().getName()`。 可以有效防止sql注入



![image-20211105111630738](E:\TyporaPic\image-20211105111630738.png)

## 2.MyBatis工作流程（原理）？

![img](https://pic1.zhimg.com/80/v2-6f49ecb1b2adaab73c93c5227d8071cc_720w.jpg?source=1940ef5c)

1. **加载mybatis全局配置文件**（数据源、mapper映射文件等），解析配置文件，MyBatis基于XML配置文件**生成Configuration，和一个个MappedStatement**（包括了参数映射配置、动态SQL语句、结果映射配置），其对应着<select | update | delete | insert>标签项
2. SqlSessionFactoryBuilder**通过Configuration对象生成SqlSessionFactory**，用来开启SqlSession。
3. SqlSession对象完成和数据库的交互：
   1. 用户程序调用mybatis接口层api（即Mapper接口中的方法）
   2. SqlSession通过**调用api的Statement ID找到对应的MappedStatement对象**
   3. 通过**Executor**(负责动态SQL的生成和查询缓存的维护)将**MappedStatement对象进行解析**，**sql参数转化，动态sql拼接，生成jdbc Statement对象**，使用**Paramterhandler**参数填充，使用**statementHandler**绑定参数。
   4. JDBC执行sql
4. 借助MappedStatement中的结果映射关系,使用**ResultSetHandler**将结果转化为**HashMap、JavaBean**等存储结构并返回。
5. 关闭sqlSession

mybatis 全局配置文件 解析配置文件 生成Configuration和一个个MappedStatement

SqlSessionsFactoryBuilder 通过Configuration对象生成 SqlSessionFactory 

工厂中获取一个SqlSession对象和数据库交互



SqlSession通过调动api的Statement id 找到对应的MappedStatement对象

通过ExeCutor将MappedStatement对象进行解析，sql参数转化，动态sql拼接，申城jdbcstatement对象 使用parameterHandler参数填充，使用statementHandler绑定参数



JDBC执行SQL 获取到ResultSetHandler 转化为HashMap、JAVAbean





### 2.1**MyBatis的主要成员**

- Configuration  MyBatis所有的**配置信息都保存在Configuration  对象中**，配置文件中的大部分配置都会存储到该类中

- SqlSession 作为MyBatis工作的主要顶层API，表示和**数据库交互时的会话，完成必要数据库增删改查功能**

- Executor  MyBatis执行器，是**MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护**

- StatementHandler  **封装了JDBC Statement操作**，负责对JDBC statement 的操作，如设置参数等

- ParameterHandler 负责对**用户传递的参数转换成JDBC Statement 所对应的数据类型**

- ResultSetHandler 负责将**JDBC返回的ResultSet结果集对象转换成List类型的集合**

- TypeHandler 负责**java数据类型和jdbc数据类型**

- **(也可以说是数据表列类型)之间的映射和转换**

- **MappedStatement**  MappedStatement**维护一条<select|update|delete|insert>节点的封装**

- SqlSource              负责**根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中**，并返回

- BoundSql              表示**动态生成的SQL语句以及相应的参数信息**

  

  以上主要成员在一次数据库操作中基本都会涉及，在SQL操作中重点需要关注的是SQL参数什么时候被设置和结果集怎么转换为JavaBean对象的，这两个过程正好对应StatementHandler和ResultSetHandler类中的处理逻辑。

  ![img](https://pic1.zhimg.com/80/v2-30f950b633174356767cce07999effb7_720w.jpg?source=1940ef5c)

  

## 3.Xml 映射文件中，除了常见的 select|insert|updae|delete 标签之外，还有哪些标签？

还有很多其他的标签，`<resultMap>`、`<parameterMap>`、`<sql>`、`<include>`、`<selectKey>`，加上动态 sql 的 9 个标签，`trim|where|set|foreach|if|choose|when|otherwise|bind`等，其中为 sql 片段标签，通过`<include>`标签引入 sql 片段，`<selectKey>`为不支持自增的主键生成策略标签。

## 4.最佳实践中，通常一个 Xml 映射文件，都会写一个 Dao 接口与之对应，请问，这个 Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？

Dao 接口，就是人们常说的 `Mapper`接口，**接口的全限名，就是映射文件中的 namespace 的值**，接口的方法名，就是映射文件中`MappedStatement`的 id 值，接口方法内的参数，就是传递给 sql 的参数。 mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可以唯一确定一个mappedstatement,举例：`com.mybatis3.mappers.StudentDao.findStudentById` 可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面`id = findStudentById`的`MappedStatement`。在 MyBatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象。

Dao 接口里的方法可以重载，但是Mybatis的XML里面的ID不允许重复。

Mybatis版本3.3.0，亲测如下：

```
/**
 * Mapper接口里面方法重载
 */
public interface StuMapper {

	List<Student> getAllStu();
    
	List<Student> getAllStu(@Param("id") Integer id);
}
```

然后在 `StuMapper.xml` 中利用Mybatis的动态sql就可以实现。

```xml
	<select id="getAllStu" resultType="com.pojo.Student">
 		select * from student
		<where>
			<if test="id != null">
				id = #{id}
			</if>
		</where>
 	</select>
```

能正常运行，并能得到相应的结果，这样就实现了在Dao接口中写重载方法。

**Mybatis 的 Dao 接口可以有多个重载方法，但是多个接口对应的映射必须只有一个，否则启动会报错。**

Dao 接口的**工作原理是 JDK 动态代理**，MyBatis 运行时会**使用 JDK 动态代理为 Dao 接口生成代理 proxy 对象，代理对象 proxy 会拦截接口方法，转而执行`MappedStatement`所代表的 sql，然后将 sql 执行结果返回**。



## 5.MyBatis 是如何进行分页的？分页插件的原理是什么？

MyBatis 使用 **RowBounds 对象进行分页**，它是**针对 ResultSet 结果集执行的内存分页，而非物理分页**，可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 **MyBatis 提供的插件接口，实现自定义插件**，在**插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数**。

举例：`select _ from student`，拦截 sql 后重写为：`select t._ from （select \* from student）t limit 0，10`



## 6.简述 MyBatis 的插件运行原理，以及如何编写一个插件。

MyBatis 仅可以编写针对 **`ParameterHandler(数据类型进行转换)`、`ResultSetHandler（JDBC返回的ResultSet转换为list对象）`、`StatementHandler（封装了JDBC的操作）`、`Executor（负责SQL语句的生成和查询缓存的维护）` 这 4 种接口的插件**，MyBatis **使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能（也就是Mybatis拦截器只能拦截这4个接口）**，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 **`InvocationHandler` 的 `invoke()`方法**，当然，**只会拦截那些你指定需要拦截的方法**。

实现 **MyBatis 的 Interceptor 接口并复写` intercept()`方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法**即可，记住，别忘了在配置文件中配置你编写的插件。



## 7.MyBatis 执行批量插入，能返回数据库主键列表吗？

能，JDBC 都能，MyBatis 当然也能。

## 8.MyBatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？

MyBatis 动态 sql 可以让我们在 Xml 映射文件内，以**标签的形式编写动态 sql**，**完成逻辑判断和动态拼接 sql 的功能**，MyBatis 提供了 9 种动态 sql 标签 `trim|where|set|foreach|if|choose|when|otherwise|bind`。

其执行原理为，**使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能**。



## 9.MyBatis 是如何将 sql 执行结果封装为目标对象并返回的？都有哪些映射形式？

第一种是使用`<resultMap>`标签，逐一定义**列名和对象属性名**之间的映射关系

第二种是使用 **sql 列的别名功能**，将列别名书写为对象属性名，比如 T_NAME AS NAME，对象属性名一般是 name，小写，但是列名不区分大小写，**MyBatis 会忽略列名大小写，智能找到与之对应对象属性名**，你甚至可以写成 T_NAME AS NaMe，MyBatis 一样可以正常工作。

有了**列名与属性名的映射关系**后，**MyBatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。**



## 10.MyBatis 能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。

关联对象查询，有两种实现方式：

1. 一种是使用**嵌套查询**，在resultMap中使用**association标签关联嵌套查询的sql语句**。
   - `<association property="way" column="wayId" javaType="com.whx.bus.entity.Way" select="selectWayById">`
2. 另一种是使用 **标签指定resultMapId**，将**关联查询的记录映射到集合List中** - **其去重复的原理是`<resultMap>`标签内的`<id>`子标签，指定了唯一确定一条记录的 id 列，MyBatis 根据列值来完成 100 条记录的去重复功能，可以有多个，代表了联合主键的语意**。





## 11.MyBatis 是否支持延迟加载？如果支持，它的实现原理是什么？

什么是延迟加载？就是在需要用到数据的时候才进行加载，不需要用到数据的时候就不加载数据。延迟加载也称为懒加载。 

MyBatis **仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association 指的就是一对一，collection 指的就是一对多查询**。在 MyBatis 配置文件中，可以配置是否启用延迟加载 `lazyLoadingEnabled=true|false。`

它的**原理**是，**使用` CGLIB` 创建目标对象的代理对象**，当调用目标方法时，进入拦截器方法，比如调用 `a.getB().getName()`，拦截器 `invoke()`方法发现 `a.getB()`是 null 值，那么就会**单独发送事先保存好的查询关联 B 对象的 sql**，把 B 查询上来后，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 `a.getB().getName()`方法的调用。这就是延迟加载的基本原理。

当然了，不光是 MyBatis，几乎所有的包括 Hibernate，支持延迟加载的原理都是一样的。



## 12.MyBatis 的 Xml 映射文件中，不同的 Xml 映射文件，id 是否可以重复？

不同的 Xml 映射文件，如果配置了 namespace，那么 id 可以重复；如果没有配置 namespace，那么 id 不能重复；毕竟 namespace 不是必须的，只是最佳实践而已。

原因就是 **namespace+id 是作为 `Map<String, MappedStatement>`的 key 使用的**，如果没有 namespace，就剩下 id，那么，id 重复会导致数据互相覆盖。有了 namespace，自然 id 就可以重复，namespace 不同，namespace+id 自然也就不同。



## 13.MyBatis 中如何执行批处理？

使用 BatchExecutor 完成批处理。



## 14.MyBatis 都有哪些 Executor 执行器？它们之间的区别是什么？

MyBatis 有三种基本的 Executor 执行器，**`SimpleExecutor`、`ReuseExecutor`、`BatchExecutor`。**

**`SimpleExecutor`：**每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。

**``ReuseExecutor`：**执行 update 或 select，以 **sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内**，供下一次使用。简言之，**就是重复使用 Statement 对象**。

**`BatchExecutor`：**执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），**它缓存了多个 Statement 对象**，**每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。**

作用范围：Executor 的这些特点，都**严格限制在 SqlSession 生命周期范围内**。

### 14.1MyBatis 中如何指定使用哪一种 Executor 执行器？

在 MyBatis 配置文件中，可以指定默认的 ExecutorType 执行器类型，也可以手动给 `DefaultSqlSessionFactory` 的创建 SqlSession 的方法传递 ExecutorType 类型参数。

## 15.MyBatis 是否可以映射 Enum 枚举类？

MyBatis可以映射枚举类，不单可以映射枚举类，MyBatis 可以**映射任何对象到表的一列上**，映射方式为自定义一个 **`TypeHandler`**，实现 `TypeHandler` 的 `setParameter()`和 `getResult()`接口方法。`TypeHandler` 有两个作用，一是完成从 javaType 至 jdbcType 的转换，二是完成 jdbcType 至 javaType 的转换，体现为 `setParameter()`和 `getResult()`两个方法，分别代表**设置 sql 问号占位符参数**和**获取列查询结果**。



## 16.MyBatis 映射文件中，如果 A 标签通过 include 引用了 B 标签的内容，请问，B 标签能否定义在 A 标签的后面，还是说必须定义在 A 标签的前面？

虽然 MyBatis 解析 Xml 映射文件是按照顺序解析的，但是，被引用的 B 标签依然可以定义在任何地方，MyBatis 都可以正确识别。

原理是，MyBatis 解析 A 标签，发现 A 标签引用了 B 标签，但是 B 标签尚未解析到，尚不存在，此时，MyBatis 会将 A 标签标记为未解析状态，然后继续解析余下的标签，包含 B 标签，待所有标签解析完毕，MyBatis 会重新解析那些被标记为未解析的标签，此时再解析 A 标签时，B 标签已经存在，A 标签也就可以正常解析完成了。



## 17.简述 MyBatis 的 Xml 映射文件和 MyBatis 内部数据结构之间的映射关系？

MyBatis 将所有 **Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部**。在 Xml 映射文件中，**`<parameterMap>`标签会被解析为 `ParameterMap` 对象**，其**每个子元素会被解析为 ParameterMapping 对象**。**`<resultMap>`标签会被解析为 `ResultMap` 对象，其每个子元素会被解析为 `ResultMapping` 对象**。每一个**`<select>、<insert>、<update>、<delete>`标签均会被解析为 `MappedStatement` 对象**，标签内的 **sql 会被解析为 BoundSql 对象**。



## 18.为什么说 MyBatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？

Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而 MyBatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。



## 19.MyBatis一级缓存、二级缓存

**一级缓存的作用域是SQlSession**, **Mybatis默认开启一级缓存**。 在同一个SqlSession中，执行相同的SQL查询时；第一次会去查询数据库，并写在缓存中，第二次会直接从缓存中取。 当执行SQL时候两次查询中间发生了增删改的操作，则SQLSession的缓存会被清空。

- Mybatis的**内部缓存使用一个HashMap，key为Statement Id + Offset + Limit + Sql + Params语句。**Value为查询出来的结果集映射成的java对象。 SqlSession执行insert、update、delete等操作**commit后会清空该SQLSession缓存**。
  - ① select * from table limit 2,1; //含义是跳过2条取出1条数据
  - ② select * from table limit 2 offset 1; //含义是从第1条数据开始取出2条数据,limit后面跟的是2条数据,offset后面是从第1条开始读取,即读取第2,3条。
- 一级缓存脏读：对于不同的sqlsession A与B, A做update操作，只能刷新A自己的一级缓存，无法刷新B的一级缓存。所以，如果A与B操作同一条记录，就会有脏读。
  - 一级缓存最大范围是SqlSession内部，**有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement**。



**二级缓存作用域是Mapper级别的**，默认是没有开启。MyBatis的二级缓存相对于一级缓存来说，**实现了SqlSession之间缓存数据的共享，同时粒度更加的细，能够到namespace级别**。

- 二级缓存脏读：**在两个不同的mapper中都涉及到同一个表的增删改查操作**，当其中一个mapper对这张表进行查询操作，此时另一个mapper进行了更新操作刷新缓存，然后第一个mapper又查询了一次，那么这次查询出的数据是脏数据。
  - MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
- mybatis：查询时，**先进行二级缓存执行流程后，就会进入一级缓存的执行流程**。（mapper级别的缓存可能是其他旧的sqlsession更新的，自己的sqlsession若存在缓存，则为较新的缓存。）

总结：在分布式环境下，**由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据**



## 20.MyBatis的设计模式

- Builder模式：例如SqlSessionFactoryBuilder、XMLConfigBuilder、XMLMapperBuilder、XMLStatementBuilder、CacheBuilder；

  - > SqlSessionFactoryBuilder会调用XMLConfigBuilder读取所有的MybatisMapConfig.xml和所有的*Mapper.xml文件，构建Mybatis运行的核心对象Configuration对象，然后将该Configuration对象作为参数构建一个SqlSessionFactory对象。

- 模板方法模式：例如BaseExecutor和SimpleExecutor，还有BaseTypeHandler和所有的子类例如IntegerTypeHandler；

- 工厂模式，例如SqlSessionFactory、ObjectFactory、MapperProxyFactory；

- 单例模式，例如ErrorContext和LogFactory；

- 代理模式，Mybatis**实现的核心**，比如MapperProxy、ConnectionLogger，**用的jdk的动态代理**；还有executor.loader包使用了cglib或者javassist达到延迟加载的效果；

- 适配器模式，例如Log的Mybatis接口和它对jdbc、log4j等各种日志框架的适配实现；

- 组合模式，例如SqlNode和各个子类ChooseSqlNode等；