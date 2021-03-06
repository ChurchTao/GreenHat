

[![](http://opyzduw4a.bkt.clouddn.com/TIM%E6%88%AA%E5%9B%BE20170929155836.png)](https://github.com/ChurchTao/green-hat)

[Quick Start]()&nbsp; | &nbsp;[Demo Project](https://github.com/ChurchTao/greenhatdemo)&nbsp; | &nbsp;[English]()

![Build](https://img.shields.io/badge/Java-1.6+-green.svg?style=flat-square)
![Version 1.1](https://img.shields.io/badge/version-1.1-yellow.svg?style=flat-square)


## 当前版本更新:

## What is green-hat ?

green-hat 是一款轻量级MVC框架。沿用Spring的注解风格(暂时也只能是注解...)，在学习spring原理的途中

想着自己造一个简单的轮子。于是就有了这个

如果你喜欢它请为它 [Star](https://github.com/ChurchTao/green-hat/stargazers) 谢谢

## 特性

* [x] MVC框架，不依赖更多的库
* [x] Spring注解风格
* [x] 支持html | jsp | json 输出
* [x] 已经支持 AOP
* [x] 已经支持 ORM
* [x] 强力JDBC支持,方法名解析,动态实现抽象方法

## 概述

* 简洁的：框架设计简单,容易理解,不依赖于更多第三方库。你完全可以看懂框架里面的简单代码。
* 优雅的：`green-hat` 沿用Spring的注解风格，方便记忆。

## 快速入门 [demo](https://github.com/ChurchTao/greenhatdemo)

# 这Demo很老了，我建议还是看 [新项目](https://github.com/FogiveHat/w-paper/tree/newVersion)
# 注意是newVersion分支

新建maven工程，在pom.xml中加入`Maven` 配置：

```xml
<dependency>
    <groupId>com.github.churchtao</groupId>
    <artifactId>greenhat</artifactId>
    <version>1.2</version>
</dependency>
```

接下来在resources目录下创建 `config.properties` 用来存放配置信息，除了包地址必须，其他都选填[有默认值]

```properties
app.base_package=com.test       项目包地址[必须]

app.jsp_path=/WEB-INF/view/     Jsp目录       [默认 /WEB-INF/view/]
app.asset_path=/asset/          资源目录      [默认 /asset/]
app.home_page=/index.html       主页          [默认 /index.html]
app.www_path=/www/              html页面地址  [默认 /www/]
app.upload_limit=10             上传文件大小  [默认10mb]
#支持跨域
allowCORS=true

#jdbc
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/book?useUnicode=true&characterEncoding=UTF-8
jdbc.type=mysql                 [sqlserver|oracle] 
jdbc.username=root
jdbc.password=root
printSQL=true                    [true|false]
```

## API示例

```java
    @Autowired
    private UserService userService;  //自动注入

    @Mapping(value = "/index",method = RequestMethod.get)
    public View index(){
        //man= BeanHelper.getBean(Man.class);
        man.setName("church");
        DataContext.Session.put("loginUser",man);
        return new View("index.jsp");
    }
    
    @Mapping(value = "/index2",method = RequestMethod.get)
    public String index(Param param){
        userService.out();
        Man man2 = DataContext.Session.get("loginUser");
        man2.say();
        return "index2.html";
    }
```

## 上传文件 and 表单参数

```java
    @Mapping(value = "/upload",method = RequestMethod.post)
    public Data uploadFile(Param param, HttpServletRequest request){
        //获取表单数据
        Map<String,Object> formMap = param.getFieldMap();
        //获取上传文件
        FileParam fileParam = param.getFile("photo");
        //保存文件  -- uploadFile() 如果上传文件想存在项目目录内，请使用 2 或 3
        //1
        UploadHelper.uploadFile("/tmp/upload/",fileParam);
        //2
        //UploadHelper.uploadFile(DataContext.getRequest().getSession().getServletContext().getRealPath("/tmp/upload/"),fileParam);
        //3
        //UploadHelper.uploadFile(request.getSession().getServletContext().getRealPath("/tmp/upload/"),fileParam);
        return new Data(fileParam.getFileName()+" "+fileParam.getFileSize());
    }
```
## DAO
```java
//将类继承与BaseDAO<?>并将实体类传入
//框架就会自动提供 save delete update get 四个默认方法
@DAO
public abstract class BookDAO extends BaseDAO<Book> {
    //需要框架来解析方法名的方法都注解上 @DAOMethod
    //框架将自动根据 find  Book  By  Name 来解析动作动态生成实现类
    @DAOMethod
    public abstract List<Book> findBookByName(String name);
    //也可以直接赋予 sql 参数按顺序放在方法的入参中
    //框架也将自动执行sql返回相应的数据无需手动实现该方法
    @DAOMethod(sql = "select * from book where name=?")
    public abstract List<Book> find(String name);
    
    //当然，假如你想手动写一些查询，可以调用  Query.* 里面的方法实现
    public List<Book> findBook2(int pageNumber, int pageSize) {
            return Query.selectList(pageNumber, pageSize, Book.class, "","");
        }
    public Book findBood3(String bookname){
            return Query.select(Admin.class,"bookname=?",bookname);
        }
}
```

## Service
```java
@Service
public class BookService {
    @Autowired
    BookDAO bookDAO;
    
    public List<Book> findBookByName(String name){
        return bookDAO.findBookByName(name);
    }

    public Book get(int id){
        return bookDAO.get(id,"bookId");
    }
}
```

## Controller中调用
```java
@Controller
public class IndexController {

    @Autowired
    BookService bookService;
    
    @Mapping(value = "/conn")
    public List<Book> conn(){
        return bookService.findBookByName("鲁滨逊漂流记");
    }
    
    @Mapping(value = "/getBook")
    public Book getBook(){
        return bookService.get(1);
    }
}
```

## 写切面方法

```java
@Aspect(Controller.class)
public class ControllerAspect extends AspectProxy {
    private static final Logger logger = LoggerFactory.getLogger(ControllerAspect.class);
    private long begin;
    @Override
    public void before(Class<?> cls, Method method, Object[] params) {
        logger.info("--------begin-------");
        logger.info("class: [{}]",cls.getName());
        logger.info("method: [{}]",method.getName());
        begin =System.currentTimeMillis();
    }
    @Override
    public void after(Class<?> cls, Method method, Object[] params, Object result) {
        logger.info("time: [{}] ms",System.currentTimeMillis()-begin);
        logger.info("--------end-------");
    }
}
```
新建上述的切面类之后，在被 @Controller 注解的类中需要被代理的方法上注解 @AspectMethod
```java
 @AspectMethod
    @Mapping(value = "/index",method = RequestMethod.get)
    public View index(){
        //man= BeanHelper.getBean(Man.class);
        man.setName("church");
        DataContext.Session.put("loginUser",man);
        return new View("index.jsp");
    }
```
运行结果：[控制台log日志输出如下~]
```text
 com.greenhat.mvc.DispatcherServlet | Handled get://index
 com.greenhat.test.ControllerAspect | --------begin-------
 com.greenhat.test.ControllerAspect | class: [com.greenhat.test.IndexController]
 com.greenhat.test.ControllerAspect | method: [index]
 com.greenhat.test.ControllerAspect | time: [22] ms
 com.greenhat.test.ControllerAspect | --------end-------
```

## 联系我

- Mail: swkzymlyy#gmail.com
