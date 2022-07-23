## 总体框架设计

![](https://picture.lingzero.cn/202207231230167.png)

### 接口层

MyBatis和数据库的交互有两种方式：

- 使用传统的MyBatis提供的API；

  ![](https://picture.lingzero.cn/202207231242735.png)

- 使用Mapper接口；

![](https://picture.lingzero.cn/202207231242464.png)

### 数据处理层

![image-20220723125248631](https://picture.lingzero.cn/202207231252353.png)

### 框架支撑层

![image-20220723125341285](https://picture.lingzero.cn/202207231253700.png)

- 事务管理机制

事务管理机制对于ORM框架而言是不可缺少的一部分，事务管理机制的质量也是考量一个ORM框架是否优秀的一个标准。

- 连接池管理机制

由于创建一个数据库连接所占用的资源比较大， 对于数据吞吐量大和访问量非常大的应用而言，连接池的设计就显得非常重要。

- 缓存机制

为了提高数据利用率和减小服务器和数据库的压力，MyBatis 会对于一些查询提供会话级别的数据缓存，会将对某一次查询，放置到SqlSession 中，在允许的时间间隔内，对于完全相同的查询，MyBatis 会直接将缓存结果返回给用户，而不用再到数据库中查找。

- SQL语句的配置方式

传统的MyBatis 配置SQL 语句方式就是使用XML文件进行配置的，但是这种方式不能很好地支持面向接口编程的理念，为了支持面向接口的编程，MyBatis 引入了Mapper接口的概念，面向接口的引入，对使用注解来配置SQL 语句成为可能，用户只需要在接口上添加必要的注解即可，不用再去配置XML文件了，但是，目前的MyBatis 只是对注解配置SQL 语句提供了有限的支持，某些高级功能还是要依赖XML配置文件配置SQL 语句。

### 引导层

![image-20220723125406999](https://picture.lingzero.cn/202207231254255.png)

### 主要构件及其相互关系

![](https://picture.lingzero.cn/202207231254303.png)

主要的核心部件解释如下：

- `SqlSession` 作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能
- `Executor` MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
- `StatementHandler` 封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。
- `ParameterHandler` 负责对用户传递的参数转换成JDBC Statement 所需要的参数，
- `ResultSetHandler` 负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
- `TypeHandler` 负责java数据类型和jdbc数据类型之间的映射和转换
- `MappedStatement` MappedStatement维护了一条`<select|update|delete|insert>`节点的封装，
- `SqlSource` 负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
- `BoundSql` 表示动态生成的SQL语句以及相应的参数信息
- `Configuration` MyBatis所有的配置信息都维持在Configuration对象之中。

### MyBatis 初始化基本过程

#### 基于XML配置文件

```java
// mybatis初始化
String resource = "mybatis-config.xml";  
InputStream inputStream = Resources.getResourceAsStream(resource);  
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

// 创建SqlSession
SqlSession sqlSession = sqlSessionFactory.openSession();  

// 执行SQL语句
List list = sqlSession.selectList("com.foo.bean.BlogMapper.queryAllBlogInfo");

```

##### 初始化的基本过程

![](https://picture.lingzero.cn/202207231258277.png)



- 调用SqlSessionFactoryBuilder对象的build(inputStream)方法；
- SqlSessionFactoryBuilder会根据输入流inputStream等信息创建XMLConfigBuilder对象;
- SqlSessionFactoryBuilder调用XMLConfigBuilder对象的parse()方法；
- XMLConfigBuilder对象返回Configuration对象；
- SqlSessionFactoryBuilder根据Configuration对象创建一个DefaultSessionFactory对象；
- SqlSessionFactoryBuilder返回 DefaultSessionFactory对象给Client，供Client使用。

SqlSessionFactoryBuilder相关的代码如下所示：

```java
public SqlSessionFactory build(InputStream inputStream)  {  
    return build(inputStream, null, null);  
}  

public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties)  {  
    try  {  
        //2. 创建XMLConfigBuilder对象用来解析XML配置文件，生成Configuration对象  
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);  
        //3. 将XML配置文件内的信息解析成Java对象Configuration对象  
        Configuration config = parser.parse();  
        //4. 根据Configuration对象创建出SqlSessionFactory对象  
        return build(config);  
    } catch (Exception e) {  
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);  
    } finally {  
        ErrorContext.instance().reset();  
        try {  
            inputStream.close();  
        } catch (IOException e) {  
            // Intentionally ignore. Prefer previous error.  
        }  
    }
}

// 从此处可以看出，MyBatis内部通过Configuration对象来创建SqlSessionFactory,用户也可以自己通过API构造好Configuration对象，调用此方法创SqlSessionFactory  
public SqlSessionFactory build(Configuration config) {  
    return new DefaultSqlSessionFactory(config);  
}  
  
    
    
```



上述的初始化过程中，涉及到了以下几个对象：

- SqlSessionFactoryBuilder ： SqlSessionFactory的构造器，用于创建SqlSessionFactory，采用了Builder设计模式
- Configuration ：该对象是mybatis-config.xml文件中所有mybatis配置信息
- SqlSessionFactory：SqlSession工厂类，以工厂形式创建SqlSession对象，采用了Factory工厂设计模式
- XmlConfigParser ：负责将mybatis-config.xml配置文件解析成Configuration对象，共SqlSessonFactoryBuilder使用，创建SqlSessionFactory



##### 创建Configuration对象的过程

> 接着上述的 MyBatis初始化基本过程讨论，当SqlSessionFactoryBuilder执行build()方法，调用了XMLConfigBuilder的parse()方法，然后返回了Configuration对象。那么parse()方法是如何处理XML文件，生成Configuration对象的呢？

- **XMLConfigBuilder会将XML配置文件的信息转换为Document对象**

而XML配置定义文件DTD转换成XMLMapperEntityResolver对象，然后将二者封装到XpathParser对象中，XpathParser的作用是提供根据Xpath表达式获取基本的DOM节点Node信息的操作。如下图所示：

![](https://picture.lingzero.cn/202207231301458.png)

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/framework/orm-mybatis/mybatis-y-init.html

- **之后XMLConfigBuilder调用parse()方法**

会从XPathParser中取出`<configuration>`节点对应的Node对象，然后解析此Node节点的子Node：properties, settings, typeAliases,typeHandlers, objectFactory, objectWrapperFactory, plugins, environments,databaseIdProvider, mappers：

```java
public Configuration parse() {  
    if (parsed) {  
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");  
    }  
    parsed = true;  
    //源码中没有这一句，只有 parseConfiguration(parser.evalNode("/configuration"));  
    //为了让读者看得更明晰，源码拆分为以下两句  
    XNode configurationNode = parser.evalNode("/configuration");  
    parseConfiguration(configurationNode);  
    return configuration;  
}  
/** 
 * 解析 "/configuration"节点下的子节点信息，然后将解析的结果设置到Configuration对象中 
 */  
private void parseConfiguration(XNode root) {  
    try {  
        //1.首先处理properties 节点     
        propertiesElement(root.evalNode("properties")); //issue #117 read properties first  
        //2.处理typeAliases  
        typeAliasesElement(root.evalNode("typeAliases"));  
        //3.处理插件  
        pluginElement(root.evalNode("plugins"));  
        //4.处理objectFactory  
        objectFactoryElement(root.evalNode("objectFactory"));  
        //5.objectWrapperFactory  
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));  
        //6.settings  
        settingsElement(root.evalNode("settings"));  
        //7.处理environments  
        environmentsElement(root.evalNode("environments")); // read it after objectFactory and objectWrapperFactory issue #631  
        //8.database  
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));  
        //9.typeHandlers  
        typeHandlerElement(root.evalNode("typeHandlers"));  
        //10.mappers  
        mapperElement(root.evalNode("mappers"));  
    } catch (Exception e) {  
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);  
    }  
} 
  
       
    
```



注意：在上述代码中，还有一个非常重要的地方，就是解析XML配置文件子节点`<mappers>`的方法mapperElements(root.evalNode("mappers")), 它将解析我们配置的Mapper.xml配置文件，Mapper配置文件可以说是MyBatis的核心，MyBatis的特性和理念都体现在此Mapper的配置和设计上。

- **然后将这些值解析出来设置到Configuration对象中**

解析子节点的过程这里就不一一介绍了，用户可以参照MyBatis源码仔细揣摩，我们就看上述的environmentsElement(root.evalNode("environments")); 方法是如何将environments的信息解析出来，设置到Configuration对象中的：

```java
/** 
 * 解析environments节点，并将结果设置到Configuration对象中 
 * 注意：创建envronment时，如果SqlSessionFactoryBuilder指定了特定的环境（即数据源）； 
 *      则返回指定环境（数据源）的Environment对象，否则返回默认的Environment对象； 
 *      这种方式实现了MyBatis可以连接多数据源 
 */  
private void environmentsElement(XNode context) throws Exception {  
    if (context != null)  
    {  
        if (environment == null)  
        {  
            environment = context.getStringAttribute("default");  
        }  
        for (XNode child : context.getChildren())  
        {  
            String id = child.getStringAttribute("id");  
            if (isSpecifiedEnvironment(id))  
            {  
                //1.创建事务工厂 TransactionFactory  
                TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));  
                DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));  
                //2.创建数据源DataSource  
                DataSource dataSource = dsFactory.getDataSource();  
                //3. 构造Environment对象  
                Environment.Builder environmentBuilder = new Environment.Builder(id)  
                .transactionFactory(txFactory)  
                .dataSource(dataSource);  
                //4. 将创建的Envronment对象设置到configuration 对象中  
                configuration.setEnvironment(environmentBuilder.build());  
            }  
        }  
    }  
}

private boolean isSpecifiedEnvironment(String id)  
{  
    if (environment == null)  
    {  
        throw new BuilderException("No environment specified.");  
    }  
    else if (id == null)  
    {  
        throw new BuilderException("Environment requires an id attribute.");  
    }  
    else if (environment.equals(id))  
    {  
        return true;  
    }  
    return false;  
} 
  
       
    
```



- **返回Configuration对象**

将上述的MyBatis初始化基本过程的序列图细化：

![](https://picture.lingzero.cn/202207231302895.png)



#### 初始化方式 - 基于Java API

当然我们可以使用XMLConfigBuilder手动解析XML配置文件来创建Configuration对象，代码如下：

```java
String resource = "mybatis-config.xml";  
InputStream inputStream = Resources.getResourceAsStream(resource);  
// 手动创建XMLConfigBuilder，并解析创建Configuration对象  
XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, null,null); // 看这里 
Configuration configuration = parser.parse();  
// 使用Configuration对象创建SqlSessionFactory  
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);  
// 使用MyBatis  
SqlSession sqlSession = sqlSessionFactory.openSession();  
List list = sqlSession.selectList("com.foo.bean.BlogMapper.queryAllBlogInfo");  
  
```

## SQLSession执行流程

### SQLSessionFactory 与 SQLSession

> 通过前面的章节对于mybatis 的介绍及使用，大家都能体会到SqlSession的重要性了吧，没错，从表面上来看，咱们都是通过SqlSession去执行sql语句（注意：是从表面看，实际的待会儿就会讲）。

正如其名，Sqlsession对应着一次数据库会话。由于数据库会话不是永久的，因此Sqlsession的生命周期也不应该是永久的，相反，在你每次访问数据库时都需要创建它（当然并不是说在Sqlsession里只能执行一次sql，你可以执行多次，当一旦关闭了Sqlsession就需要重新创建它）。

那么咱们就先看看是怎么获取SqlSession的吧：

![](https://picture.lingzero.cn/202207231306889.png)

首先，SqlSessionFactoryBuilder去读取mybatis的配置文件，然后build一个DefaultSqlSessionFactory。源码如下：

```java
 /**
  * 一系列的构造方法最终都会调用本方法（配置文件为Reader时会调用本方法，还有一个InputStream方法与此对应）
  * @param reader
  * @param environment
  * @param properties
  * @return
  */
 public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
   try {
     //通过XMLConfigBuilder解析配置文件，解析的配置相关信息都会封装为一个Configuration对象
     XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
     //这儿创建DefaultSessionFactory对象
     return build(parser.parse());
   } catch (Exception e) {
     throw ExceptionFactory.wrapException("Error building SqlSession.", e);
   } finally {
     ErrorContext.instance().reset();
     try {
       reader.close();
     } catch (IOException e) {
       // Intentionally ignore. Prefer previous error.
     }
   }
 }

 public SqlSessionFactory build(Configuration config) {
   return new DefaultSqlSessionFactory(config);
 }
  
        
```

当我们获取到SqlSessionFactory之后，就可以通过SqlSessionFactory去获取SqlSession对象。源码如下：

```java
/**
  * 通常一系列openSession方法最终都会调用本方法
  * @param execType 
  * @param level
  * @param autoCommit
  * @return
  */
 private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
   Transaction tx = null;
   try {
     //通过Confuguration对象去获取Mybatis相关配置信息, Environment对象包含了数据源和事务的配置
     final Environment environment = configuration.getEnvironment();
     final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
     tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
     //之前说了，从表面上来看，咱们是用sqlSession在执行sql语句， 实际呢，其实是通过excutor执行， excutor是对于Statement的封装
     final Executor executor = configuration.newExecutor(tx, execType);
     //关键看这儿，创建了一个DefaultSqlSession对象
     return new DefaultSqlSession(configuration, executor, autoCommit);
   } catch (Exception e) {
     closeTransaction(tx); // may have fetched a connection so lets call close()
     throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
   } finally {
     ErrorContext.instance().reset();
   }
 }

```



通过以上步骤，咱们已经得到SqlSession对象了。接下来就是该干嘛干嘛去了（话说还能干嘛，当然是执行sql语句咯）。看了上面，咱们也回想一下之前写的Demo：

```java
SqlSessionFactory sessionFactory = null;  
String resource = "mybatis-conf.xml";  
try {
    //SqlSessionFactoryBuilder读取配置文件
   sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader(resource));
} catch (IOException e) {  
   e.printStackTrace();  
}    
//通过SqlSessionFactory获取SqlSession
SqlSession sqlSession = sessionFactory.openSession();
  
       
    
```



创建Sqlsession的地方只有一个，那就是SqlsessionFactory的openSession方法：

```java
public SqlSessionopenSession() {  
    return openSessionFromDataSource(configuration.getDefaultExecutorType(),null, false);  
}
  
    
```



我们可以看到实际创建SqlSession的地方是openSessionFromDataSource，如下：

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {  
 
    Connection connection = null;  
 
    try {  
 
        final Environment environment = configuration.getEnvironment();  
 
        final DataSource dataSource = getDataSourceFromEnvironment(environment);  
        
        // MyBatis对事务的处理相对简单，TransactionIsolationLevel中定义了几种隔离级别，并不支持内嵌事务这样较复杂的场景，同时由于其是持久层的缘故，所以真正在应用开发中会委托Spring来处理事务实现真正的与开发者隔离。分析事务的实现是个入口，借此可以了解不少JDBC规范方面的事情。
        TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);  
 
        connection = dataSource.getConnection();  
 
        if (level != null) {  
            connection.setTransactionIsolation(level.getLevel());
        }  
 
        connection = wrapConnection(connection);  
 
        Transaction tx = transactionFactory.newTransaction(connection,autoCommit);  
 
        Executorexecutor = configuration.newExecutor(tx, execType);  
 
        return newDefaultSqlSession(configuration, executor, autoCommit);  
 
    } catch (Exceptione) {  
        closeConnection(connection);  
        throwExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);  
    } finally {
        ErrorContext.instance().reset();
    }
}  
  
      
    
```

可以看出，创建sqlsession经过了以下几个主要步骤：

- 从配置中获取Environment；
- 从Environment中取得DataSource；
- 从Environment中取得TransactionFactory；
- 从DataSource里获取数据库连接对象Connection；
- 在取得的数据库连接上创建事务对象Transaction；
- 创建Executor对象（该对象非常重要，事实上sqlsession的所有操作都是通过它完成的）；
- 创建sqlsession对象。

SqlSession咱们也拿到了，咱们可以调用SqlSession中一系列的select..., insert..., update..., delete...方法轻松自如的进行CRUD操作了。就这样？那咱配置的映射文件去哪儿了？别急，咱们接着往下看。

### 利器之MapperProxy

![](https://picture.lingzero.cn/202207231308506.png)

在mybatis中，通过MapperProxy动态代理咱们的dao， 也就是说， 当咱们执行自己写的dao里面的方法的时候，其实是对应的mapperProxy在代理。那么，咱们就看看怎么获取MapperProxy对象吧：

通过SqlSession从Configuration中获取。源码如下：

```java
 /**
  * 什么都不做，直接去configuration中找， 哥就是这么任性
  */
 @Override
 public <T> T getMapper(Class<T> type) {
   return configuration.<T>getMapper(type, this);
 }
SqlSession把包袱甩给了Configuration, 接下来就看看Configuration。源码如下：
 /**
  * 烫手的山芋，俺不要，你找mapperRegistry去要
  * @param type
  * @param sqlSession
  * @return
  */
 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
   return mapperRegistry.getMapper(type, sqlSession);
 }
Configuration不要这烫手的山芋，接着甩给了MapperRegistry， 那咱看看MapperRegistry。 源码如下：
 /**
  * 烂活净让我来做了，没法了，下面没人了，我不做谁来做
  * @param type
  * @param sqlSession
  * @return
  */
 @SuppressWarnings("unchecked")
 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
   //能偷懒的就偷懒，俺把粗活交给MapperProxyFactory去做
   final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
   if (mapperProxyFactory == null) {
     throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
   }
   try {
     //关键在这儿
     return mapperProxyFactory.newInstance(sqlSession);
   } catch (Exception e) {
     throw new BindingException("Error getting mapper instance. Cause: " + e, e);
   }
 }
  
       
    
```




MapperProxyFactory是个苦B的人，粗活最终交给它去做了。咱们看看源码：

```java
 /**
  * 别人虐我千百遍，我待别人如初恋
  * @param mapperProxy
  * @return
  */
 @SuppressWarnings("unchecked")
 protected T newInstance(MapperProxy<T> mapperProxy) {
   //动态代理我们写的dao接口
   return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
 }
 
 public T newInstance(SqlSession sqlSession) {
   final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
   return newInstance(mapperProxy);
 }
  
       
    
```



通过以上的动态代理，咱们就可以方便地使用dao接口啦， 就像之前咱们写的demo那样：

```java
UserDao userMapper = sqlSession.getMapper(UserDao.class);  
User insertUser = new User();
  
       
    
```



这下方便多了吧， 呵呵， 貌似mybatis的源码就这么一回事儿啊。具体详细介绍，请参见MyBatis Mapper 接口如何通过JDK动态代理来包装SqlSession 源码分析。别急，还没完， 咱们还没看具体是怎么执行sql语句的呢。

### Excutor

![](https://picture.lingzero.cn/202207231309703.png)



- **Executor创建的源代码**：

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {  

    executorType = executorType == null ? defaultExecutorType : executorType;  

    executorType = executorType == null ?ExecutorType.SIMPLE : executorType;  

    Executor executor;  

    if(ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this,transaction);
    } else if(ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this,transaction);  
    } else {  
        executor = newSimpleExecutor(this, transaction);
    }

    if (cacheEnabled) {
        executor = new CachingExecutor(executor);  
    }
    executor = (Executor) interceptorChain.pluginAll(executor);  
    return executor;  
}  
  
        
    
```

可以看出，如果不开启cache的话，创建的Executor只是3中基础类型之一，BatchExecutor专门用于执行批量sql操作，ReuseExecutor会重用statement执行sql操作，SimpleExecutor只是简单执行sql没有什么特别的。开启cache的话（默认是开启的并且没有任何理由去关闭它），就会创建CachingExecutor，它以前面创建的Executor作为唯一参数。CachingExecutor在查询数据库前先查找缓存，若没找到的话调用delegate（就是构造时传入的Executor对象）从数据库查询，并将查询结果存入缓存中。

Executor对象是可以被插件拦截的，如果定义了针对Executor类型的插件，最终生成的Executor对象是被各个插件插入后的代理对象。

接下来，咱们才要真正去看sql的执行过程了。上面，咱们拿到了MapperProxy, 每个MapperProxy对应一个dao接口， 那么咱们在使用的时候，MapperProxy是怎么做的呢？ 源码奉上：

- **MapperProxy**

我们知道对被代理对象的方法的访问都会落实到代理者的invoke上来，MapperProxy的invoke如下：

```java
  /**
   * MapperProxy在执行时会触发此方法
   */
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (Object.class.equals(method.getDeclaringClass())) {
      try {
        return method.invoke(this, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    //二话不说，主要交给MapperMethod自己去管
    return mapperMethod.execute(sqlSession, args);
  }
  
        
    
```



- **MapperMethod**

就像是一个分发者，他根据参数和返回值类型选择不同的sqlsession方法来执行。这样mapper对象与sqlsession就真正的关联起来了。

```java
  /**
   * 看着代码不少，不过其实就是先判断CRUD类型，然后根据类型去选择到底执行sqlSession中的哪个方法，绕了一圈，又转回sqlSession了
   * @param sqlSession
   * @param args
   * @return
   */
  public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    if (SqlCommandType.INSERT == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
    } else if (SqlCommandType.UPDATE == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
    } else if (SqlCommandType.DELETE == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
    } else if (SqlCommandType.SELECT == command.getType()) {
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
      } else if (method.returnsMany()) {
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
      } else {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
      }
    } else {
      throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName() 
          + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }
  

    
```




既然又回到SqlSession了，前面提到过，sqlsession只是一个门面，真正发挥作用的是executor，对sqlsession方法的访问最终都会落到executor的相应方法上去。Executor分成两大类，一类是CacheExecutor，另一类是普通Executor。Executor的创建前面已经介绍了，那么咱们就看看SqlSession的CRUD方法了，为了省事，还是就选择其中的一个方法来做分析吧。这儿，咱们选择了selectList方法：

```java
  public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
      MappedStatement ms = configuration.getMappedStatement(statement);
      //CRUD实际上是交给Excetor去处理， excutor其实也只是穿了个马甲而已，小样，别以为穿个马甲我就不认识你嘞！
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
  
      
    
```



- **CacheExecutor**

CacheExecutor有一个重要属性delegate，它保存的是某类普通的Executor，值在构照时传入。执行数据库update操作时，它直接调用delegate的update方法，执行query方法时先尝试从cache中取值，取不到再调用delegate的查询方法，并将查询结果存入cache中。代码如下：

```java
public List query(MappedStatement ms, Object parameterObject, RowBounds rowBounds,ResultHandler resultHandler) throws SQLException {  
    if (ms != null) {  
        Cache cache = ms.getCache();  
        if (cache != null) {  
            flushCacheIfRequired(ms);  
            cache.getReadWriteLock().readLock().lock();  
            try {  
                if (ms.isUseCache() && resultHandler ==null) {  
                    CacheKey key = createCacheKey(ms, parameterObject, rowBounds);  
                    final List cachedList = (List)cache.getObject(key);  
                    if (cachedList != null) {  
                        return cachedList;  
                    } else {  
                        List list = delegate.query(ms,parameterObject, rowBounds, resultHandler);  
                        tcm.putObject(cache,key, list);  
                        return list;  
                    }  
                } else {  
                    return delegate.query(ms,parameterObject, rowBounds, resultHandler);  
                }  
            } finally {  
                cache.getReadWriteLock().readLock().unlock();  
            }
        }  
    }  
    return delegate.query(ms,parameterObject, rowBounds, resultHandler);  
}
  
       
    
```



- **普通Executor**

有3类，他们都继承于BaseExecutor，BatchExecutor专门用于执行批量sql操作，ReuseExecutor会重用statement执行sql操作，SimpleExecutor只是简单执行sql没有什么特别的。下面以SimpleExecutor为例：

```java
public List doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds,ResultHandler resultHandler) throws SQLException {  
    Statement stmt = null;  
    try {  
        Configuration configuration = ms.getConfiguration();  
        StatementHandler handler = configuration.newStatementHandler(this, ms,parameter, rowBounds,resultHandler);  
        stmt =prepareStatement(handler);  
        returnhandler.query(stmt, resultHandler);  
    } finally {  
        closeStatement(stmt);  
    }  
} 
  
    
    
```



然后，通过一层一层的调用，最终会来到doQuery方法， 这儿咱们就随便找个Excutor看看doQuery方法的实现吧，我这儿选择了SimpleExecutor:

```java
  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      stmt = prepareStatement(handler, ms.getStatementLog());
      //StatementHandler封装了Statement, 让 StatementHandler 去处理
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }
   
    
```



Mybatis内置的ExecutorType有3种，默认的是simple，该模式下它为每个语句的执行创建一个新的预处理语句，单条提交sql；而batch模式重复使用已经预处理的语句， 并且批量执行所有更新语句，显然batch性能将更优；

但batch模式也有自己的问题，比如在Insert操作时，在事务没有提交之前，是没有办法获取到自增的id，这在某型情形下是不符合业务要求的；

通过走码和研读spring相关文件发现，在同一事务中batch模式和simple模式之间无法转换，由于本项目一开始选择了simple模式，所以碰到需要批量更新时，只能在单独的事务中进行；

在代码中使用batch模式可以使用以下方式：

```java
//从spring注入原有的sqlSessionTemplate
@Autowired
private SqlSessionTemplate sqlSessionTemplate;
 
public void testInsertBatchByTrue() {
    //新获取一个模式为BATCH，自动提交为false的session
    //如果自动提交设置为true,将无法控制提交的条数，改为最后统一提交，可能导致内存溢出
    SqlSession session = sqlSessionTemplate.getSqlSessionFactory().openSession(ExecutorType.BATCH, false);
    //通过新的session获取mapper
    fooMapper = session.getMapper(FooMapper.class);
    int size = 10000;
    try {
        for (int i = 0; i < size; i++) {
            Foo foo = new Foo();
            foo.setName(String.valueOf(System.currentTimeMillis()));
            fooMapper.insert(foo);
            if (i % 1000 == 0 || i == size - 1) {
                //手动每1000个一提交，提交后无法回滚
                session.commit();
                //清理缓存，防止溢出
                session.clearCache();
            }
        }
    } catch (Exception e) {
        //没有提交的数据可以回滚
        session.rollback();
    } finally {
        session.close();
    }
}
  
   
    
```

上述代码没有使用spring的事务，改手动控制，如果和原spring事务一起使用，将无法回滚，必须注意，最好单独使用；

##  一级缓存实现机制

### MyBatis一级缓存实现

#### 什么是一级缓存？ 为什么使用一级缓存？

每当我们使用MyBatis开启一次和数据库的会话，MyBatis会创建出一个SqlSession对象表示一次数据库会话。

在对数据库的一次会话中，我们有可能会反复地执行完全相同的查询语句，如果不采取一些措施的话，每一次查询都会查询一次数据库,而我们在极短的时间内做了完全相同的查询，那么它们的结果极有可能完全相同，由于查询一次数据库的代价很大，这有可能造成很大的资源浪费。

为了解决这一问题，减少资源的浪费，MyBatis会在表示会话的SqlSession对象中建立一个简单的缓存，将每次查询到的结果结果缓存起来，当下次查询的时候，如果判断先前有个完全一样的查询，会直接从缓存中直接将结果取出，返回给用户，不需要再进行一次数据库查询了。

如下图所示，MyBatis一次会话: 一个SqlSession对象中创建一个本地缓存(local cache)，对于每一次查询，都会尝试根据查询的条件去本地缓存中查找是否在缓存中，如果在缓存中，就直接从缓存中取出，然后返回给用户；否则，从数据库读取数据，将查询结果存入缓存并返回给用户。

![](https://picture.lingzero.cn/202207231316312.png)

对于会话（Session）级别的数据缓存，我们称之为一级数据缓存，简称一级缓存。



> 即SqlSession中的缓存是怎样组织的？由于MyBatis使用SqlSession对象表示一次数据库的会话，那么，对于会话级别的一级缓存也应该是在SqlSession中控制的。

实际上, MyBatis只是一个MyBatis对外的接口，SqlSession将它的工作交给了Executor执行器这个角色来完成，负责完成对数据库的各种操作。当创建了一个SqlSession对象时，MyBatis会为这个SqlSession对象创建一个新的Executor执行器，而缓存信息就被维护在这个Executor执行器中，MyBatis将缓存和对缓存相关的操作封装成了Cache接口中。SqlSession、Executor、Cache之间的关系如下列类图所示：

![](https://picture.lingzero.cn/202207231317040.png)



如上述的类图所示，Executor接口的实现类BaseExecutor中拥有一个Cache接口的实现类PerpetualCache，则对于BaseExecutor对象而言，它将使用PerpetualCache对象维护缓存。

综上，SqlSession对象、Executor对象、Cache对象之间的关系如下图所示：

![](https://picture.lingzero.cn/202207231317592.png)

由于Session级别的一级缓存实际上就是使用PerpetualCache维护的，那么PerpetualCache是怎样实现的呢？

PerpetualCache实现原理其实很简单，其内部就是通过一个简单的`HashMap<k,v>` 来实现的，没有其他的任何限制。如下是PerpetualCache的实现代码：

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/framework/orm-mybatis/mybatis-y-cache-level1.html

```java
package org.apache.ibatis.cache.impl;  
  
import java.util.HashMap;  
import java.util.Map;  
import java.util.concurrent.locks.ReadWriteLock;  
  
import org.apache.ibatis.cache.Cache;  
import org.apache.ibatis.cache.CacheException;  
  
/** 
 * 使用简单的HashMap来维护缓存 
 * @author Clinton Begin 
 */  
public class PerpetualCache implements Cache {  
  
  private String id;  
  
  private Map<Object, Object> cache = new HashMap<Object, Object>();  
  
  public PerpetualCache(String id) {  
    this.id = id;  
  }  
  
  public String getId() {  
    return id;  
  }  
  
  public int getSize() {  
    return cache.size();  
  }  
  
  public void putObject(Object key, Object value) {  
    cache.put(key, value);  
  }  
  
  public Object getObject(Object key) {  
    return cache.get(key);  
  }  
  
  public Object removeObject(Object key) {  
    return cache.remove(key);  
  }  
  
  public void clear() {  
    cache.clear();  
  }  
  
  public ReadWriteLock getReadWriteLock() {  
    return null;  
  }  
  
  public boolean equals(Object o) {  
    if (getId() == null) throw new CacheException("Cache instances require an ID.");  
    if (this == o) return true;  
    if (!(o instanceof Cache)) return false;  
  
    Cache otherCache = (Cache) o;  
    return getId().equals(otherCache.getId());  
  }  
  
  public int hashCode() {  
    if (getId() == null) throw new CacheException("Cache instances require an ID.");  
    return getId().hashCode();  
  }  
  
} 
  
    
    
```



####  一级缓存的生命周期有多长？



MyBatis在开启一个数据库会话时，会创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象，Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。

- 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用；
- 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用；
- SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用；

![](https://picture.lingzero.cn/202207231318928.png)



#### SqlSession 一级缓存的工作流程

- 对于某个查询，根据statementId,params,rowBounds来构建一个key值，根据这个key值去缓存Cache中取出对应的key值存储的缓存结果；
- 判断从Cache中根据特定的key值取的数据数据是否为空，即是否命中；
- 如果命中，则直接将缓存结果返回；
- 如果没命中：
  - 去数据库中查询数据，得到查询结果；
  - 将key和查询到的结果分别作为key,value对存储到Cache中；
  - 将查询结果返回；
- 结束。

![](https://picture.lingzero.cn/202207231319678.png)



#### Cache接口的设计以及CacheKey的定义

如下图所示，MyBatis定义了一个org.apache.ibatis.cache.Cache接口作为其Cache提供者的SPI(Service Provider Interface) ，所有的MyBatis内部的Cache缓存，都应该实现这一接口。MyBatis定义了一个PerpetualCache实现类实现了Cache接口，实际上，在SqlSession对象里的Executor对象内维护的Cache类型实例对象，就是PerpetualCache子类创建的。

（MyBatis内部还有很多Cache接口的实现，一级缓存只会涉及到这一个PerpetualCache子类，Cache的其他实现将会放到二级缓存中介绍）。

![](https://picture.lingzero.cn/202207231319995.png)



我们知道，Cache最核心的实现其实就是一个Map，将本次查询使用的特征值作为key，将查询结果作为value存储到Map中。现在最核心的问题出现了：怎样来确定一次查询的特征值？换句话说就是：怎样判断某两次查询是完全相同的查询？也可以这样说：如何确定Cache中的key值？

MyBatis认为，对于两次查询，如果以下条件都完全一样，那么就认为它们是完全相同的两次查询：

- 传入的 statementId
- 查询时要求的结果集中的结果范围 （结果的范围通过rowBounds.offset和rowBounds.limit表示）
- 这次查询所产生的最终要传递给JDBC java.sql.Preparedstatement的Sql语句字符串（boundSql.getSql() ）
- 传递给java.sql.Statement要设置的参数值

**现在分别解释上述四个条件**：

- 传入的statementId，对于MyBatis而言，你要使用它，必须需要一个statementId，它代表着你将执行什么样的Sql；
- MyBatis自身提供的分页功能是通过RowBounds来实现的，它通过rowBounds.offset和rowBounds.limit来过滤查询出来的结果集，这种分页功能是基于查询结果的再过滤，而不是进行数据库的物理分页；
- 由于MyBatis底层还是依赖于JDBC实现的，那么，对于两次完全一模一样的查询，MyBatis要保证对于底层JDBC而言，也是完全一致的查询才行。而对于JDBC而言，两次查询，只要传入给JDBC的SQL语句完全一致，传入的参数也完全一致，就认为是两次查询是完全一致的。
- 上述的第3个条件正是要求保证传递给JDBC的SQL语句完全一致；第4条则是保证传递给JDBC的参数也完全一致；即3、4两条MyBatis最本质的要求就是：调用JDBC的时候，传入的SQL语句要完全相同，传递给JDBC的参数值也要完全相同。

综上所述,CacheKey由以下条件决定：**statementId + rowBounds + 传递给JDBC的SQL + 传递给JDBC的参数值**；

- **CacheKey的创建**

对于每次的查询请求，Executor都会根据传递的参数信息以及动态生成的SQL语句，将上面的条件根据一定的计算规则，创建一个对应的CacheKey对象。

我们知道创建CacheKey的目的，就两个：

- 根据CacheKey作为key,去Cache缓存中查找缓存结果；
- 如果查找缓存命中失败，则通过此CacheKey作为key，将从数据库查询到的结果作为value，组成key,value对存储到Cache缓存中；

CacheKey的构建被放置到了Executor接口的实现类BaseExecutor中，定义如下：

```java
/** 
 * 所属类:  org.apache.ibatis.executor.BaseExecutor 
 * 功能   :   根据传入信息构建CacheKey 
 */  
public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {  
    if (closed) throw new ExecutorException("Executor was closed.");  
    CacheKey cacheKey = new CacheKey();  
    //1.statementId  
    cacheKey.update(ms.getId());  
    //2. rowBounds.offset  
    cacheKey.update(rowBounds.getOffset());  
    //3. rowBounds.limit  
    cacheKey.update(rowBounds.getLimit());  
    //4. SQL语句  
    cacheKey.update(boundSql.getSql());  
    //5. 将每一个要传递给JDBC的参数值也更新到CacheKey中  
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();  
    TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();  
    for (int i = 0; i < parameterMappings.size(); i++) { // mimic DefaultParameterHandler logic  
        ParameterMapping parameterMapping = parameterMappings.get(i);  
        if (parameterMapping.getMode() != ParameterMode.OUT) {  
            Object value;  
            String propertyName = parameterMapping.getProperty();  
            if (boundSql.hasAdditionalParameter(propertyName)) {  
                value = boundSql.getAdditionalParameter(propertyName);  
            } else if (parameterObject == null) {  
                value = null;  
            } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {  
                value = parameterObject;  
            } else {  
                MetaObject metaObject = configuration.newMetaObject(parameterObject);  
                value = metaObject.getValue(propertyName);  
            }  
            //将每一个要传递给JDBC的参数值也更新到CacheKey中  
            cacheKey.update(value);  
        }  
    }  
    return cacheKey;  
}
  
        
    
```



- **CacheKey的hashcode生成算法**

刚才已经提到，Cache接口的实现，本质上是使用的`HashMap<k,v>`,而构建CacheKey的目的就是为了作为`HashMap<k,v>`中的key值。而HashMap是通过key值的hashcode 来组织和存储的，那么，构建CacheKey的过程实际上就是构造其hashCode的过程。下面的代码就是CacheKey的核心hashcode生成算法，感兴趣的话可以看一下：

```java
public void update(Object object) {  
    if (object != null && object.getClass().isArray()) {  
        int length = Array.getLength(object);  
        for (int i = 0; i < length; i++) {  
            Object element = Array.get(object, i);  
            doUpdate(element);  
        }  
    } else {  
        doUpdate(object);  
    }  
}  
 
private void doUpdate(Object object) {  
 
    //1. 得到对象的hashcode;    
    int baseHashCode = object == null ? 1 : object.hashCode();  
    //对象计数递增  
    count++;  
    checksum += baseHashCode;  
    //2. 对象的hashcode 扩大count倍  
    baseHashCode *= count;  
    //3. hashCode * 拓展因子（默认37）+拓展扩大后的对象hashCode值  
    hashcode = multiplier * hashcode + baseHashCode;  
    updateList.add(object);  
} 
  
        
    
```

MyBatis认为的完全相同的查询，不是指使用sqlSession查询时传递给算起来Session的所有参数值完完全全相同，你只要保证statementId，rowBounds,最后生成的SQL语句，以及这个SQL语句所需要的参数完全一致就可以了。

####  一级缓存的性能分析

- **MyBatis对会话（Session）级别的一级缓存设计的比较简单，就简单地使用了HashMap来维护，并没有对HashMap的容量和大小进行限制**

读者有可能就觉得不妥了：如果我一直使用某一个SqlSession对象查询数据，这样会不会导致HashMap太大，而导致 java.lang.OutOfMemoryError错误啊？ 读者这么考虑也不无道理，不过MyBatis的确是这样设计的。

MyBatis这样设计也有它自己的理由：

- 一般而言SqlSession的生存时间很短。一般情况下使用一个SqlSession对象执行的操作不会太多，执行完就会消亡；
- 对于某一个SqlSession对象而言，只要执行update操作（update、insert、delete），都会将这个SqlSession对象中对应的一级缓存清空掉，所以一般情况下不会出现缓存过大，影响JVM内存空间的问题；
- 可以手动地释放掉SqlSession对象中的缓存。
- **一级缓存是一个粗粒度的缓存，没有更新缓存和缓存过期的概念**

MyBatis的一级缓存就是使用了简单的HashMap，MyBatis只负责将查询数据库的结果存储到缓存中去， 不会去判断缓存存放的时间是否过长、是否过期，因此也就没有对缓存的结果进行更新这一说了。

根据一级缓存的特性，在使用的过程中，我认为应该注意：

- 对于数据变化频率很大，并且需要高时效准确性的数据要求，我们使用SqlSession查询的时候，要控制好SqlSession的生存时间， SqlSession的生存时间越长，它其中缓存的数据有可能就越旧，从而造成和真实数据库的误差；同时对于这种情况，用户也可以手动地适时清空SqlSession中的缓存；
- 对于只执行、并且频繁执行大范围的select操作的SqlSession对象，SqlSession对象的生存时间不应过长。

## 二级缓存实现机制

### MyBatis二级缓存实现

MyBatis的二级缓存是Application级别的缓存，它可以提高对数据库查询的效率，以提高应用的性能。

#### MyBatis的缓存机制整体设计以及二级缓存的工作模式

![](https://picture.lingzero.cn/202207231322225.png)

如图所示，当开一个会话时，一个SqlSession对象会使用一个Executor对象来完成会话操作，MyBatis的二级缓存机制的关键就是对这个Executor对象做文章。如果用户配置了"cacheEnabled=true"，那么MyBatis在为SqlSession对象创建Executor对象时，会对Executor对象加上一个装饰者：CachingExecutor，这时SqlSession使用CachingExecutor对象来完成操作请求。CachingExecutor对于查询请求，会先判断该查询请求在Application级别的二级缓存中是否有缓存结果，如果有查询结果，则直接返回缓存结果；如果缓存中没有，再交给真正的Executor对象来完成查询操作，之后CachingExecutor会将真正Executor返回的查询结果放置到缓存中，然后在返回给用户。

![](https://picture.lingzero.cn/202207231322051.png)



CachingExecutor是Executor的装饰者，以增强Executor的功能，使其具有缓存查询的功能，这里用到了设计模式中的`装饰者模式`，CachingExecutor和Executor的接口的关系如下类图所示：

![](https://picture.lingzero.cn/202207231322507.png)



#### MyBatis二级缓存的划分

MyBatis并不是简单地对整个Application就只有一个Cache缓存对象，它将缓存划分的更细，即是Mapper级别的，即每一个Mapper都可以拥有一个Cache对象，具体如下：

- **为每一个Mapper分配一个Cache缓存对象**（使用`<cache>`节点配置）

MyBatis将Application级别的二级缓存细分到Mapper级别，即对于每一个Mapper.xml,如果在其中使用了`<cache>` 节点，则MyBatis会为这个Mapper创建一个Cache缓存对象，如下图所示：

![](https://picture.lingzero.cn/202207231323857.png)



注：上述的每一个Cache对象，都会有一个自己所属的namespace命名空间，并且会将Mapper的 namespace作为它们的ID；

- **多个Mapper共用一个Cache缓存对象**（使用`<cache-ref>`节点配置）

如果你想让多个Mapper公用一个Cache的话，你可以使用`<cache-ref namespace="">`节点，来指定你的这个Mapper使用到了哪一个Mapper的Cache缓存。

![](https://picture.lingzero.cn/202207231323603.png)



#### 使用二级缓存，必须要具备的条件

MyBatis对二级缓存的支持粒度很细，它会指定某一条查询语句是否使用二级缓存。

虽然在Mapper中配置了`<cache>`,并且为此Mapper分配了Cache对象，这并不表示我们使用Mapper中定义的查询语句查到的结果都会放置到Cache对象之中，我们必须指定Mapper中的某条选择语句是否支持缓存，即如下所示，在`<select>` 节点中配置useCache="true"，Mapper才会对此Select的查询支持缓存特性，否则，不会对此Select查询，不会经过Cache缓存。如下所示，Select语句配置了useCache="true"，则表明这条Select语句的查询会使用二级缓存。

```xml
<select id="selectByMinSalary" resultMap="BaseResultMap" parameterType="java.util.Map" useCache="true">
  
        
    
```



总之，要想使某条Select查询支持二级缓存，你需要保证：

- MyBatis支持二级缓存的总开关：全局配置变量参数 cacheEnabled=true
- 该select语句所在的Mapper，配置了`<cache>` 或`<cached-ref>`节点，并且有效
- 该select语句的参数 useCache=true



#### 一级缓存和二级缓存的使用顺序

请注意，如果你的MyBatis使用了二级缓存，并且你的Mapper和select语句也配置使用了二级缓存，那么在执行select查询的时候，MyBatis会先从二级缓存中取输入，其次才是一级缓存，

即**MyBatis查询数据的顺序是：二级缓存 ———> 一级缓存 ——> 数据库**。

#### 二级缓存实现的选择

MyBatis对二级缓存的设计非常灵活，它自己内部实现了一系列的Cache缓存实现类，并提供了各种缓存刷新策略如LRU，FIFO等等；另外，MyBatis还允许用户自定义Cache接口实现，用户是需要实现org.apache.ibatis.cache.Cache接口，然后将Cache实现类配置在`<cache type="">`节点的type属性上即可；除此之外，MyBatis还支持跟第三方内存缓存库如Memecached的集成，总之，使用MyBatis的二级缓存有三个选择:

- MyBatis自身提供的缓存实现；
- 用户自定义的Cache接口实现；
- 跟第三方内存缓存库的集成；



#### MyBatis自身提供的二级缓存的实现

> MyBatis自身提供了丰富的，并且功能强大的二级缓存的实现，它拥有一系列的Cache接口装饰者，可以满足各种对缓存操作和更新的策略。

MyBatis定义了大量的Cache的装饰器来增强Cache缓存的功能，如下类图所示。

![](https://picture.lingzero.cn/202207231326960.png)

对于每个Cache而言，都有一个容量限制，MyBatis各供了各种策略来对Cache缓存的容量进行控制，以及对Cache中的数据进行刷新和置换。MyBatis主要提供了以下几个刷新和置换策略：

- LRU：（Least Recently Used）,最近最少使用算法，即如果缓存中容量已经满了，会将缓存中最近最少被使用的缓存记录清除掉，然后添加新的记录；
- FIFO：（First in first out）,先进先出算法，如果缓存中的容量已经满了，那么会将最先进入缓存中的数据清除掉；
- Scheduled：指定时间间隔清空算法，该算法会以指定的某一个时间间隔将Cache缓存中的数据清空；



### 如何细粒度地控制你的MyBatis二级缓存

#### [¶](#一个关于mybatis的二级缓存的实际问题) 一个关于MyBatis的二级缓存的实际问题

现有AMapper.xml中定义了对数据库表 ATable 的CRUD操作，BMapper定义了对数据库表BTable的CRUD操作；

假设 MyBatis 的二级缓存开启，并且 AMapper 中使用了二级缓存，AMapper对应的二级缓存为ACache；

除此之外，AMapper 中还定义了一个跟BTable有关的查询语句，类似如下所述：

```xml
<select id="selectATableWithJoin" resultMap="BaseResultMap" useCache="true">  
      select * from ATable left join BTable on ....  
</select>
  
        
    
```



执行以下操作：

- 执行AMapper中的"selectATableWithJoin" 操作，此时会将查询到的结果放置到AMapper对应的二级缓存ACache中；
- 执行BMapper中对BTable的更新操作(update、delete、insert)后，BTable的数据更新；
- 再执行1完全相同的查询，这时候会直接从AMapper二级缓存ACache中取值，将ACache中的值直接返回；

好，**问题就出现在第3步**上：

由于AMapper的“selectATableWithJoin” 对应的SQL语句需要和BTable进行join查找，而在第 2 步BTable的数据已经更新了，但是第 3 步查询的值是第 1 步的缓存值，已经极有可能跟真实数据库结果不一样，即ACache中缓存数据过期了！

总结来看，就是：

对于某些使用了 join连接的查询，如果其关联的表数据发生了更新，join连接的查询由于先前缓存的原因，导致查询结果和真实数据不同步；

从MyBatis的角度来看，这个问题可以这样表述：

**对于某些表执行了更新(update、delete、insert)操作后，如何去清空跟这些表有关联的查询语句所造成的缓存**

#### [¶](#当前mybatis二级缓存的工作机制) 当前MyBatis二级缓存的工作机制

> MyBatis二级缓存的一个重要特点：即松散的Cache缓存管理和维护

![](https://picture.lingzero.cn/202207231327486.png)

一个Mapper中定义的增删改查操作只能影响到自己关联的Cache对象。如上图所示的Mapper namespace1中定义的若干CRUD语句，产生的缓存只会被放置到相应关联的Cache1中，即Mapper namespace2,namespace3,namespace4 中的CRUD的语句不会影响到Cache1。

可以看出，**Mapper之间的缓存关系比较松散，相互关联的程度比较弱**。

现在再回到上面描述的问题，如果我们将AMapper和BMapper共用一个Cache对象，那么，当BMapper执行更新操作时，可以清空对应Cache中的所有的缓存数据，这样的话，数据不是也可以保持最新吗？

确实这个也是一种解决方案，不过，它会使缓存的使用效率变的很低！AMapper和BMapper的任意的更新操作都会将共用的Cache清空，会频繁地清空Cache，导致Cache实际的命中率和使用率就变得很低了，所以这种策略实际情况下是不可取的。

最理想的解决方案就是：

**对于某些表执行了更新(update、delete、insert)操作后，去清空跟这些指定的表有关联的查询语句所造成的缓存**; 这样，就是以很细的粒度管理MyBatis内部的缓存，使得缓存的使用率和准确率都能大大地提升。