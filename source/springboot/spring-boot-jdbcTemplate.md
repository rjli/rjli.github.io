# JDBC Template 的使用

## 数据源配置

在我们访问数据库的时候，需要先配置一个数据源，下面分别介绍一下几种不同的数据库配置方式。

首先，为了连接数据库需要引入jdbc支持，在`pom.xml`中引入如下配置：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

#### 嵌入式数据库支持

嵌入式数据库通常用于开发和测试环境，不推荐用于生产环境。Spring Boot提供自动配置的嵌入式数据库有H2、HSQL、Derby，你不需要提供任何连接配置就能使用。

比如，我们可以在`pom.xml`中引入如下配置使用HSQL

```
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
```

#### 连接生产数据源

以MySQL数据库为例，先引入MySQL连接的依赖包，在`pom.xml`中加入：

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.21</version>
</dependency>
```

在`src/main/resources/application.properties`中配置数据源信息

```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

#### 连接JNDI数据源

当你将应用部署于应用服务器上的时候想让数据源由应用服务器管理，那么可以使用如下配置方式引入JNDI数据源。

```
spring.datasource.jndi-name=java:jboss/datasources/customers
```



## 使用JDBCTemplate

在数据库中创建一张User表（id，name,age）,其中id设置为自增长类型的。

创建一个UserService接口类，实现基本的增，删，改，查方法：

```
public interface UserService{
    /**
     * 新增一个用户
     * @param name
     * @param age
     */
    void create(String name, Integer age);

    /**
     * 根据name删除一个用户高
     * @param name
     */
    void deleteByName(String name);

    /**
     * 获取用户总量
     */
    Integer getAllUsers();

    /**
     * 删除所有用户
     */
    void deleteAllUsers();
}

```

通过JdbcTemplate实现UserService中定义的数据访问操作创建对应的实现类

```
@Service
public class UserServiceImpl implements UserService{
    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * 创建用户
     * @param name
     * @param age
     */
    @Override
    public void create(String name, Integer age) {
        jdbcTemplate.update("insert into user(name,age) values(?,?)",name,age);
    }

    /**
     * 根据名字删除用户
     * @param name
     */
    @Override
    public void deleteByName(String name) {
        jdbcTemplate.update("delete from user where name = ? ",name);
    }

    /**
     * 获取所有用户
     * @return
     */
    @Override
    public Integer getAllUsers() {
       return jdbcTemplate.queryForObject("select count(1) from user",Integer.class);
    }

    /**
     * 删除用户
     */
    @Override
    public void deleteAllUsers() {
        jdbcTemplate.update("delete from user");
    }
}

```

- 创建对UserService的单元测试用例，通过创建、删除和查询来验证数据库操作的正确性。

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Before
    public void setUp() {
        // 准备，清空user表
        userService.deleteAllUsers();
    }

    @Test
    public void test(){
        userService.create("张三",20);
        userService.create("李四",10);
        userService.create("王五",30);
        userService.create("赵六",42);
        userService.create("孙二",66);

        // 查数据库，应该有5个用户
        Assert.assertEquals(5, userService.getAllUsers().intValue());
        userService.deleteByName("张三");
        // 查数据库，应该有5个用户
        Assert.assertEquals(4, userService.getAllUsers().intValue());
    }
}

```



其余详细的操作见[jdbcTemplateApi](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html)

