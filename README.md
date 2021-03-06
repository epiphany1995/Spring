

# 1、spring简介

spring理念：使现有的技术更加容易使用，本身是一个大杂烩，整合了现有的技术框架。

```xml
<!-- maven对spring的依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency>
```



# 2、优点及拓展 

- spring是一个开源的免费框架（容器）。

- spring是一个轻量级的、非入侵式的框架。

- 控制反转（IOC），面向切面编程（AOP）。

- 支持事务的控制，对框架整合的支持。

  

Spring Boot 

- 一个快速开发的脚手架。
- 基于Spring Boot可以快速的开发单个微服务。
- 预定大于配置

Spring Cloud 

- Spring Cloud是基于Spring Boot实现的。

  

# 3、ioc理论推导

1. 传统模式：程序是主动创建，控制权在程序员手上。

```java
	//service层
	private UserDao userDao; 

	@Override
	public void getUser() {
		userDao = new UserDaoImpl();
		userDao.getUser();
	}
```

```java
	//main方法
	public static void main(String[] args) {
		UserService userService = new UserServiceImpl();
		userService.getUser();
	}
```

2.IOC原型：使用了set注入后，程序不再具有主动性，而是变成了被动的接受对象。

```java
	//service层
	private UserDao userDao;

	//利用set进行动态实现值得注入
	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}

	@Override
	public void getUser() {
		userDao.getUser();
	}
```

```java
//main方法
public static void main(String[] args) {
	UserService userService = new UserServiceImpl();
    //利用set方法，把接 口的实现类让用户选择，实现控制反转
    userService.setUserDao(new UserDaoImpl());

    userService.getUser();
    }  
```

这种思想，从本质上解决了问题，我们程序员不用再去管理对象的创建。系统的耦合性大大降低，可以更加专注在业务的实现上！这就是IOC的原型。

**控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是IOC容器，其实现方式是依赖注入（Dependency Injetion，DI）**



# 4、xml实现方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--  使用Spring来创建对象,在Spring这些都称为bean 
	等同于： UserDao userDao = new UserDaoImpl();
	      id : 变量名          
	      class : new的对象
	      property : 相当于给对象中的属性赋值
    -->
    <bean id="userDao" class="com.chengchao11.dao.UserDaoImpl">
    </bean>
    
    <!-- 
			ref : 引用Spring容器中创建好的对象
			value : 具体的值,基本数据类型
 	 -->
    <bean id="userService" class="com.chengchao11.service.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
        <property name="name" value="chengchao002"/>
    </bean>
</beans>
```

```java
public static void main(String[] args) {
		//获取Spring的上下文对象，对象在这一步被创建！
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
		//我们的对象现在都再Spring中管理了，我们要使用，直接去里面取！
		UserService userService = (UserService) context.getBean("userService");
		userService.getUser();
	}
```

**到现在，我们彻底不用在程序中去改动了，要实现不同的操作，只需要在xml配置文件中进行修改，所谓的IOC就是：对象由Spring来创建，管理，装配！**

## 4.1  IOC容器创建对象的方式

```java
public class UserServiceImpl implements UserService {
	
	private String name;
	
	private UserDao userDao;
	
	public UserServiceImpl() {
		System.out.println("调用UserService 无参构造器");
	}
	
	public UserServiceImpl(String name) {
		this.name = name;
		System.out.println("调用UserService 有参构造器name被赋值为:"+this.name);
	}
	
	public void setName(String name) {
		this.name = name;
		System.out.println("调用setName方法,name被赋值为:"+this.name);
	}
	
	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
		System.out.println("调用setUserDao方法");
	}

	@Override
	public void getUser() {
		userDao.getUser();
		System.out.println("UserServiceImpl.name:"+name);
	}
}

```

```java
//控制台输出:
调用UserService 无参构造器
调用setUserDao方法
调用setName方法,name被赋值为:chengchao002
默认获取用户的数据！
UserServiceImpl.name:chengchao002
```

1. **使用set方法给对象属性赋值！**

2. **使用无参构造器创建对象，默认！**

3. **假设我们要使用有参构造创建对象（三种方式）。**

   ```xml
   <!--第一种，下标赋值！-->
   <bean id="userService" class="com.chengchao11.service.UserServiceImpl">
      <constructor-arg index="0" value="chengchao001"/>
      <property name="userDao" ref="userDao"/>
      <property name="name" value="chengchao002"/>
   </bean>
   ```

   ```xml
   <!--第二种，通过类型赋值，不建议使用！-->
   <bean id="userService" class="com.chengchao11.service.UserServiceImpl">
      <constructor-arg type="java.lang.String" value="chengchao001"/>
      <property name="userDao" ref="userDao"/>
      <property name="name" value="chengchao002"/>
   </bean>
   ```

   ```xml
   <!--第三种，直接通过参数名来设置！-->
   <bean id="userService" class="com.chengchao11.service.UserServiceImpl">
      <constructor-arg name="name" value="chengchao001"/>
      <property name="userDao" ref="userDao"/>
      <property name="name" value="chengchao002"/>
   </bean>
   ```

   ```java
   //控制台输出:
   调用UserService 有参构造器，name被赋值为:chengchao001
   调用setUserDao方法
   调用setName方法，name被赋值为:chengchao002
   默认获取用户的数据！
   UserServiceImpl.name:chengchao002
   ```


## 4.2 Bean的自动装配

- 自动装配是Spring满足bean依赖的一种方式！

- Spring会在上下文中自动寻找，并自动给bean装配属性！

在Spring中有两种自动装配的方式

```xml
<!-- 
byName:会自动在容器上下文中查找，和自己对象set方法后面的值对应的beanid！
-->
<bean id="userService" class="com.chengchao11.service.UserServiceImpl" autowire="byName">
</bean>
```

```xml
<!-- 
byType:会自动在容器上下文中查找，和自己对象属性相同的beanid！
-->
<bean id="userService" class="com.chengchao11.service.UserServiceImpl" autowire="byType">
</bean>
```



# 5、注解实现方式

jdk1.5支持的注解，Spring2.5就支持注解了！

要使用注解须知：

1. 导入约束：context约束
2. 配置注解的支持：<context:annotation-config/>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
	
    <!--注解的驱动！-->
    <context:annotation-config/>

</beans>
```

## 5.1 注解实现自动装配

**@Autowired**

自动装配，直接在属性上使用即可！也可以在set方法上使用

使用@Autowired我们可以不用编写set方法了，前提是你这个自动装配的属性在 IOC 相同类型的bean。

**@Qualifier**

如果@Autowired 自动装配的环境比较复杂，自动装配无法通过一个注解完成的时候，我们可以使用

@Qualifier去配合@Autowired的使用，指定一个唯一的bean对象注入！ 



**@Autowired** 和 **@Resource**的区别：

- 都是用来实现自装配的，都可以放在属性字段上。
- **@Autowired** 是由Spring提供的注解，**@Resource**是由java提供的注解。
- **@Autowired** 通过byType的方式实现，而且必须要求这个对象存在 。
- **@Resource** 默认通过byName的方式实现，如果找不到名字，则通过byType实现，如果两个都找不到的情况下，就报错。

## 5.2 组件扫描

要使用组件扫描需要指定扫描的包。

注：context:component-scan包含了context:annotation-config，不需要重复添加注解驱动。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定要扫描的包！这个包下的注解就会生效-->
    <context:component-scan base-package="com.chengchao22"></context:component-scan>
    
</beans>
```

```java
//@Component 定义组件
//等价于 <bean id="people" class="com.chengchao22.vo.People/">
@Component
public class People {
   
   //@Value 属性值的注入
   //相当于 <property name="name" value="程超">
   @Value("chengchao11")
   private String name;
	
	@Autowired
	private Cat cat;
	
	@Autowired
	private Dog dog;

	public Cat getCat1() {
		return cat;
	}

	public Dog getDog() {
		return dog;
	}
}
```

@**Component**：放在类上定义组件，说明这个类被Spring管理了，也就是bean！

通常按照mvc三层架构有几个衍生的注解：

@**Repository**：定义dao层的组件。

@**Service**：定义serbvice层的组件。

@**controller**：定义controller层的组件。

这四个注解功能都是一样的，都是代表将某个类注册到Spring中，装配bean。



# 6、代理模式

为什么要了解代理模式？

因为代理模式就是SpringAop的顶层【重点】

代理模式的分类：

- 静态代理
- 动态代理

## 6.1、静态代理

角色分析：

- 抽象角色：一般会使用接口或者抽象类来解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，一般会做一些附属操作
- 客户：访问代理对象的人

代码步骤：

1. 接口：

   ```java
   /*
   定义一个出租房屋的接口
   在代理模式中它作为抽象角色
   */
   public interface Rend {
       public void rend() ;
   }
   ```

2. 真实角色：

   ```java
   /*
   定义一个房东类，他要出租房子
   在代理模式中它作为真实角色
   */
   public class Host implements Rend {
   
       public void rend() {
           System.out.println("房东出租房子！");
       }
   }
   ```

3. 代理角色：

   ```java
   /*
   定义一个中介类，他也要出租房子
   在代理模式中它作为代理角色
   */
   public class HostProxy implements Rend {
   
       private Host host;
   
       public HostProxy(Host host) {
           this.host = host;
       }
   
       public void rend() {
           seeHouse();
           host.rend();
           getCash();
       }
   
       public void seeHouse() {System.out.println("中介带你看房子");}
   
       public void getCash() {System.out.println("中介收中介费！");}
   }
   ```

4. 客户端访问代理角色

   ```java
   /*
   定义一个客户，他现在要租房子
   在代理模式中它作为客户端
   */
   public class Client {
       public static void main(String[] args) {
           //房东现在要出租房子
           Host host = new Host();
           //中介帮房东出租房子，但是代理一般会做一些附属操作
           HostProxy proxy = new HostProxy(host);
           //你不用找房东，直接找中介即可，房东也不用管带看房，签合同，收房租的事
           proxy.rend();
       }
   }
   ```

```java
//控制台输出:
中介带你看房子
房东出租房子！
中介收中介费！
```

代理模式的好处：

- 可以使真实角色更加纯粹！不用去关注一些公共服务
- 公共服务就交给代理角色！实现了业务的分工
- 公共业务发生拓展时，方便集中管理

缺点：

一个真实角色就会产生一个代理角色；代码量会翻倍

# 7、Spring AOP

​	**AOP意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容。利用AOP可以对业务逻辑的各个部分进行隔离，提高开发效率。**

步骤：

1. 使用AOP织入，需要导入AOP织入包

   ```xml
   <!-- spring aop织入包 -->
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.9.5</version>
   </dependency>
   ```

2. 在配置文件中添加AOP约束

   ```xml
   xmlns:aop="http://www.springframework.org/schema/aop" 
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop.xsd
   ```

3. 两种方式定义通知

   - 第一种继承MethodBeforeAdvice（前置通知）或者AfterReturningAdvice（后置通知）

     ```java
     //前置通知
     public class Logger implements MethodBeforeAdvice {
         public void before(Method method, Object[] objects, Object o) throws Throwable {
             System.out.println(method.getName()+"()方法即将被执行");
         }
     }
     ```

     ```java
     //后置通知
     public class AfterLogger implements AfterReturningAdvice {
         public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
             System.out.println(method.getName()+"()方法被执行了");
             System.out.println("返回结果为:"+o);
         }
     }
     ```

     ```xml
       <!--将通知注册到spring容器中-->
         <bean id="logger" class="com.chengchao001.utils.Logger"/>
         <bean id="afterLogger" class="com.chengchao001.utils.AfterLogger"/>
     
       <!--配置aop：首先需要导入aop的约束-->
       <!--实现aop的第一种方式-->
         <aop:config>
             <!--切入点 execution:表达式 execution(要执行的位置 * * * * *) -->
             <aop:pointcut id="pointcut" expression="execution(* com.chengchao001.service.impl.UserServiceImpl.*(..))"/>
             <!--执行环绕增加 -->
             <aop:advisor advice-ref="logger" pointcut-ref="pointcut"/>
             <aop:advisor advice-ref="afterLogger" pointcut-ref="pointcut"/>
         </aop:config>
     ```

   - 第二种方式自定义类

     ```java
      public class LoggerDiy {
     
         //自定义前置通知
         public void beforeMethd() {
             System.out.println("=======方法执行前=====");
         }
     
         //自定义后置通知
         public void afterMethd() {
             System.out.println("=======方法执行前=====");
         }
     }
     ```

     ```xml
        <!--将自定义实体类注册到spring容器中-->
         <bean id="loggerDiy" class="com.chengchao001.utils.LoggerDiy"/>
     
       <!--配置aop：首先需要导入aop的约束-->
       <!--实现aop的第二种方式-->
         <aop:config>
             <!--定义一个切面，即是我们的自定义日志类-->
             <aop:aspect ref="loggerDiy">
                 <!--切入点 execution:表达式 execution(要执行的位置 * * * * *) -->
                 <aop:pointcut id="pointcut" expression="execution(* com.chengchao001.service.impl.UserServiceImpl.*(..))"/>
                 <!--在切面中定义通知-->
                 <aop:before method="beforeMethd" pointcut-ref="pointcut"/>
                 <aop:after method="afterMethd" pointcut-ref="pointcut"/>
             </aop:aspect>
         </aop:config>
     ```

4. 直接在main方法中调用方法，查看结果

   ```java
      public static void main(String[] args) {
           ApplicationContext ctx = new 							ClassPathXmlApplicationContext("applicationContext.xml");
           //注意这里只能代理接口才能实现AOP
           UserService userService = ctx.getBean("userService", UserService.class);
   
           //添加一个用户
           UserVo user = new UserVo(3, "chengchao", 15, "chengchao@163.com");
           userService.addUser(user);
   
           //删除该用户
           userService.deleteUser(user.getUserId());
   
           //查询所有用户
           List<UserVo> userList = userService.selectUser();
       }
   ```

5. AOP实现打印日志功能![image-20200519155557838](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519155557838.png)

# 8、Spring整合Mybatis

**步骤：**

1. 导入相关jar包

   ```xml
    		<!-- spring的依赖 -->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-webmvc</artifactId>
               <version>5.2.1.RELEASE</version>
           </dependency>
   
           <!-- spring aop织入包 -->
           <dependency>
               <groupId>org.aspectj</groupId>
               <artifactId>aspectjweaver</artifactId>
               <version>1.9.5</version>
           </dependency>
   
           <!--mysql的驱动包-->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.47</version>
           </dependency>
   
           <!-- spring操作数据库的依赖包 -->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-jdbc</artifactId>
               <version>5.2.1.RELEASE</version>
           </dependency>
   
           <!--mybatis的依赖-->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.2</version>
           </dependency>
   
           <!-- spring对于mybatis的整合 -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis-spring</artifactId>
               <version>2.0.4</version>
           </dependency>
   ```

2. 编写UserDao接口

   ```java
   public interface UserDao {
   
       public List<UserVo> selectUser();
   
       public int addUser(UserVo userVo);
   
       public int deleteUser(long id);
           
   }
   ```

3. 编写mapper映射文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.chengchao001.dao.UserDao">
   
       <select id="selectUser" resultType="com.chengchao001.vo.UserVo">
           select
           USER_ID as userId,
           USER_NAME as userName,
           USER_AGE as userAge,
           USER_EMAIL as userEmail
        from vop_user_info_t;
       </select>
   
       <insert id="addUser" parameterType="com.chengchao001.vo.UserVo">
           insert into vop_user_info_t
           (
             USER_ID,USER_NAME,USER_AGE,USER_EMAIL
           ) values (
             #{userId},#{userName},#{userAge},#{userEmail}
           )
       </insert>
   
       <delete id="deleteUser" parameterType="long">
           delete from vop_user_info_t
           where USER_ID = #{id}
       </delete>
   </mapper>
   ```

4. 编写UserDaoImpl实现类

   ```java
   public class UserDaoImpl implements UserDao {
   
       private SqlSessionTemplate sqlSessions;
       
       public void setSqlSessions(SqlSessionTemplate sqlSessions) {
           UserDao mapper = sqlSessions.getMapper(UserDao.class);
           this.sqlSessions = sqlSessions;
       }
   
       public List<UserVo> selectUser() {
           UserDao mapper = sqlSessions.getMapper(UserDao.class);
           return mapper.selectUser();
       }
   
       public int addUser(UserVo userVo) {
           UserDao mapper = sqlSessions.getMapper(UserDao.class);
           return mapper.addUser(userVo);
       }
   
       public int deleteUser(long id) {
           UserDao mapper = sqlSessions.getMapper(UserDao.class);
           return mapper.deleteUser(id);
       }
   }
   ```

5. 编写Spring核心配置文件applicationContext.properties

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--使用spring的数据源-->
       <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
           <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
           <property name="url" value="jdbc:mysql://localhost:3306/mydatabase?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8"/>
           <property name="username" value="root"/>
           <property name="password" value="686995"/>
       </bean>
   
       <!--使用spring—mybatis的sqlSessionFactory-->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <property name="dataSource" ref="dataSource"/>
           <property name="mapperLocations" value="com/chengchao001/dao/UserDaoImpl.xml"/>
       </bean>
   
       <!--第一种方式使用SqlSessionTemplate，注入sqlSessionFactory-->
       <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
           <constructor-arg ref="sqlSessionFactory"/>
       </bean>
   
       <!--需要将SqlSessionTemplate的对象注入到dao的实现类中-->
       <bean id="userDao" class="com.chengchao001.dao.UserDaoImpl">
           <property name="sqlSessions" ref="sqlSession"/>
       </bean>
   
       <bean id="userService" class="com.chengchao001.service.impl.UserServiceImpl">
           <property name="userDao" ref="userDao"/>
       </bean>
   </beans>
   ```



# 9、声明式事务

**步骤**:

1. 在Spring配置文件中添加事务的约束

   ```xml
   xmlns:tx="http://www.springframework.org/schema/tx"
   http://www.springframework.org/schema/tx
   http://www.springframework.org/schema/tx/spring-tx.xsd
   ```

2. 在spring配置文件中配置声明式事务

   ```xml
      <!--配置声明式事务,注入dataSource-->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <property name="dataSource" ref="dataSource"/>
       </bean>
   ```

3. 配置事务通知

   ```xml
       <!--配置事务通知,引用上面的配置的事务-->
       <tx:advice id="txAdvice" transaction-manager="transactionManager">
           <tx:attributes>
               <!--给哪些方法配置事务 propagation:事务的七种传播特性，默认值为REQUIRED-->
               <tx:method name="add" propagation="REQUIRED"/>
               <tx:method name="delete" propagation="REQUIRED"/>
               <tx:method name="update" propagation="REQUIRED"/>
               <tx:method name="select" read-only="true"/>
           </tx:attributes>
       </tx:advice>
   ```

4. AOP实现事务通知切入

   ```xml
       <!--结合AOP实现事务织入-->
       <aop:config>
           <aop:pointcut id="pointcut" expression="execution(* com.chengchao001.service.impl.UserServiceImpl.*(..))"/>
           <!--执行环绕增加 -->
           <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
       </aop:config>
   ```

   

