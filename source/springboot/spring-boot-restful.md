#  # spring-boot restful与集成测试



- `@Controller`：修饰class，用来创建处理http请求的对象
- `@RestController`：Spring4之后加入的注解，原来在`@Controller`中返回json需要`@ResponseBody`来配合，如果直接用`@RestController`替代`@Controller`就不需要再配置`@ResponseBody`，默认返回json格式。
- `@RequestMapping`：配置url映射



下面我们尝试使用Spring MVC来实现一组对User对象操作的RESTful API，配合注释详细说明在Spring MVC中如何映射HTTP请求、如何传参、如何编写单元测试。

**RESTful API具体设计如下：**

[![img](http://blog.didispace.com/content/images/posts/springbootrestfulapi-1.png)](http://blog.didispace.com/content/images/posts/springbootrestfulapi-1.png)

## 使用Swagger生成api



