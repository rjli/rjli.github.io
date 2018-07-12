# SpringBoot ——属性配置文件

> 在SpringBoot中，在`pom.xml`中引入模块化的`Starter POMs`。其中各个模块都有自己的默认配置，所以如果不是特殊应用场景，就只需要在`application.properties`中完成一些属性配置就能开启各模块的应用。

## application.porterties文件



###  自定义属性

#### 基本使用

在Springboot中，我们可能也需要定义一些自己的使用属性。可以在application.properties中使用如下方式

```
com.cheryl.name = Cheryl
com.cheryl.password = 123456
```

然后再实体类中，使用@ConfigurationProperties(prefix = "com.cheryl")把当前前前缀下面的注解引入进来。

```
@Component
@ConfigurationProperties(prefix = "com.cheryl")
public class BlogProperties {
    private String name;
    private String password;
    //此处省略get,set方法
    }
```

然后再控制器中使用 @Autowired把实体类注入到容器中，然后启动程序在浏览器中访问对应的路径即可。

```
@RestController
@RequestMapping("/blogPropertiesController")
public class BlogPropertiesController {

    @Autowired
    private BlogProperties blogProperties;
    
    @GetMapping("/info")
    public String blogInfo(){
        return blogProperties.toString();
    }
}
```

#### 参数间的引用

在`application.properties`中的各个参数之间也可以直接引用来使用。使用‘${xxx}’的方式来引用定义过自定义属性，如下所示：

```
com.cheryl.title = "springboot技术实战"
com.cheryl.desc = ${com.cheryl.name}正在学习《${com.cheryl.title}》
```

最后该值为Cheryl正在学习《springboot技术实战》

#### 生成随机数

在有一些情况中，我们希望一些属性使用随机值而不是一个固定的值，比如密钥等。Spring Boot的属性配置文件中可以通过`${random}`来产生int值、long值或者string字符串，来支持属性的随机值。

```
# 随机字符串（32位的随机字符串）
com.cheryl.value=${random.value}
# 随机int
com.cheryl.number=${random.int}
# 随机long
com.cheryl.bignumber=${random.long}
# 10以内的随机数
com.cheryl.test1=${random.int(10)}
# 10-20的随机数
com.cheryl.test2=${random.int[10,20]}
```



## 多环境配置

我们在开发Spring Boot应用时，通常会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要修改配置文件的话，比较繁琐，而且容易出错。

一般来说多环境的配置使用各种项目构建工具或是框架的基本思路是一致的，通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外。

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：

- `application-dev.properties`：开发环境
- `application-test.properties`：测试环境
- `application-prod.properties`：生产环境

至于哪个具体的配置文件会被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。

如：`spring.profiles.active=test`就会加载`application-test.properties`配置文件内容

下面，以不同环境配置不同的服务端口为例，进行样例实验。

- 针对各环境新建不同的配置文件`application-dev.properties`、`application-test.properties`、`application-prod.properties`
- 在这三个文件均都设置不同的`server.port`属性，如：dev环境设置为82，test环境设置为83，prod环境设置为80
- application.properties中设置`spring.profiles.active=dev`，就是说默认以dev环境设置
- 测试不同配置的加载
  - 执行`java -jar xxx.jar`，可以观察到服务端口被设置为`1111`，也就是默认的开发环境（dev）
  - 执行`java -jar xxx.jar --spring.profiles.active=test`，可以观察到服务端口被设置为`2222`，也就是测试环境的配置（test）
  - 执行`java -jar xxx.jar --spring.profiles.active=prod`，可以观察到服务端口被设置为`3333`，也就是生产环境的配置（prod）

按照上面的实验，可以如下总结多环境的配置思路：

- `application.properties`中配置通用内容，并设置`spring.profiles.active=dev`，以开发环境为默认配置
- `application-{profile}.properties`中配置各个环境不同的内容
- 通过命令行方式去激活不同环境的配置

