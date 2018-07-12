# spring-data-jpa

[TOC]

## 自定义ID的生成策略

### 如何指定id策略

在JPA中，我们是通过`@id`和`@GeneratedValue`来指定id主键和id策略的，比如：

```
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
@Column(name = "id")
private String id;
```

这样也就指定了id和生成id所使用的策略，那么JPA提供了那些策略呢？

### JPA提供的4种策略

从`@GeneratedValue`源码里可以看到，`strategy`属性是由`GenerationType`指定的，而在`GenerationType`中定义了四种策略： 
- **TABLE**：使用一个特定的数据库表格来保存主键。 
- **SEQUENCE**：根据底层数据库的序列来生成主键，条件是数据库支持序列。 
- **IDENTITY**：主键由数据库自动生成（主要是自动增长型）
- **AUTO**：主键由程序控制(也是默认的,在指定主键时，如果不指定主键生成策略，默认为AUTO) 

  这些策略也不是所有数据库都支持的，具体情况如下：

| 策略\数据库 | mysql  | oracle | postgreSQL | kingbase |
| ----------- | ------ | ------ | ---------- | -------- |
| TABLE       | 支持   | 支持   | 支持       | 支持     |
| SEQUENCE    | 不支持 | 支持   | 支持       | 支持     |
| IDENTITY    | 支持   | 不支持 | 支持       | 支持     |
| AUTO        | 支持   | 支持   | 支持       | 支持     |

在`@GeneratedValue` 中还有一个generator属性



### Hibernate拓展id策略

当然，很多时候，这么几种策略并不够用，这里hibernate也拓展了JPA的id策略，我们可以在`org.hibernate.id.IdentifierGeneratorFactory`中看到，主要提供了这么些策略： 
1. **native**: 对于oracle采用Sequence方式，对于MySQL和SQL Server采用identity(自增主键生成机制)，native就是将主键的生成工作交由数据库完成，hibernate不管(很常用)。 
2. **uuid**: 采用128位的uuid算法生成主键，uuid被编码为一个32位16进制数字的字符串。占用空间大(字符串类型)。 
3. **hilo**: 使用hilo生成策略，要在数据库中建立一张额外的表，默认表名为hibernate_unique_key,默认字段为Integer类型，名称是next_hi(比较少用)。 
4. **assigned**: 在插入数据的时候主键由程序处理(很常用)，这是`generator`元素没有指定时的默认生成策略。等同于JPA中的AUTO。 
5. **identity**: 使用SQL Server和MySQL的自增字段，这个方法不能放到Oracle中，Oracle不支持自增字段，要设定sequence(MySQL和SQL Server中很常用)。等同于JPA中的IDENTITY。 
6. **select**: 使用触发器生成主键(主要用于早期的数据库主键生成机制，少用)。 
7. **sequence**: 调用底层数据库的序列来生成主键，要设定序列名，不然hibernate无法找到。 
8. **seqhilo**: 通过hilo算法实现，但是主键历史保存在Sequence中，适用于支持Sequence的数据库，如Oracle(比较少用)。 
9. **increment**: 插入数据的时候hibernate会给主键添加一个自增的主键，但是一个hibernate实例就维护一个计数器，所以在多个实例运行的时候不能使用这个方法。 
10. **foreign**: 使用另外一个相关联的对象的主键。通常和联合起来使用。 
11. **guid**: 采用数据库底层的guid算法机制，对应MYSQL的uuid()函数，SQL Server的newid()函数，ORACLE的rawtohex(sys_guid())函数等。 
12. **uuid.hex**: 看uuid，建议用uuid替换。 
13. **sequence-identity**: sequence策略的扩展，采用立即检索策略来获取sequence值，需要JDBC3.0和JDK4以上（含1.4）版本 。 

具体使用就是多了一个`@GenericGenerator`注解，指定自定义名称以及策略，然后在`@GeneratedValue`中使用该策略，比如：

```
 @Id
 @GeneratedValue(generator  = "comnIdStrategy")
 @GenericGenerator(name = "comnIdStrategy", strategy = "uuid")
 private String id;
```

###  使用自定义的id策略

hibernate 提供一种实现自定义id策略的接口`IdentifierGenerator`（位于org.hibernate.id中）。因此在生成自定义策略时，我们只需实现一下IdentifierGenerator接口，以及对应的generate方法即可：

```
public class ExpId implements IdentifierGenerator{
   ...
   @Override
    public Serializable generate(SessionImplementor sessionImplementor, Object o) throws HibernateException {
        String date = String.valueOf(new Date().getTime());
        return "exp" + date+"_"+generateRandom();
    }
}
```

然后对对应的实体类的某个字段上面使用该策略即可，`@GenericGenerator`注解的`strategy`属性上说了，使用非默认策略的时候，需要使用全类名，即：

```
@Id
@GeneratedValue(generator  = "expIdStrategy")
@GenericGenerator(name = "expIdStrategy", strategy = "com.cheryl.learn.idworker.ExpId")
@Column(name = "id")
private String id;
```

使用测试方法测试，可以看到在数据库中添加的数据的id是使用我们定义策略生成的。
|	id|address	|balance	|gender 	|name 	|password 	|
| ----------- | ------ | ------ | ---------- | -------- |---------|
|exp1524189929776_13653	|山西太原|	5000.5|	男	|张三丰	|123456|
|exp1524189929853_22297	|山西晋中|	2000.1|	男	|李四	|123456|

