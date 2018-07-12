# java web应用的http请求改成https


> 本文通过将简单介绍两种将http请求修改为https的方式。
>
> 1、通过使用sun公司提供的keytool工具结合tomcat实现
>
> 2、使用阿里云的免费证书结合nginx实现。



## 使用SUN公司的提供的工具配置证书

> SUN公司提供了制作证书的工具keytool。 在JDK 1.4以后的版本中都包含了这一工具，它的位置为<JAVA_HOME>\bin\keytool.exe。

### 生成证书

> 这里只介绍简单的keytool生成证书的方法，如果想要了解更多关于keytool的知识，请参考 [ keytool用法总结](http://ln-ydc.iteye.com/blog/1335213)


 1.创建证书,在命令行中输入

```
keytool -genkeypair -alias "test1" -keyalg "RSA" -keystore "test.keystore"  
```

![img](/images/https/1.jpg)



- 功能：

>  创建一个别名为test1的证书条目，该条目存放在名为test.keystore的密钥库中，若test.keystore密钥库不存在则创建。

- 参数说明：

> -genkeypair：生成一对非对称密钥;
>
> -alias：指定密钥对的别名，该别名是公开的;
>
> -keyalg：指定加密算法，本例中的采用通用的RAS加密算法;
>
> -keystore:密钥库的路径及名称，不指定的话，默认在操作系统的用户目录下生成一个".keystore"的文件

- 注意：

> 1.“名字与姓氏”应该是域名，若输成了姓名，和真正运行的时候域名不符，会出问题;
>
> 2.再次输入密码，第一次输入的是密钥库(keystore)的密码，第二次输入的是证书条目的密码
>
> 3.这里所说的证书库和密钥库是等同的(个人观点)

 2.查看证书库

```
keytool -list -keystore test.keystore  
```

![img](/images/https/2.jpg)

- 功能：

> 查看名为test.keystore的证书库中的证书条目

 3.导出到证书文件

```
keytool -export -alias test1 -file test.crt -keystore test.keystore  
```

![img](/images/https/3.jpg)

- 功能：

>  将名为test.keystore的证书库中别名为test1的证书条目导出到证书文件test.crt中

 4.导入证书的信息

```
keytool -import -keystore test_cacerts -file test.crt   
```

 ![img](/images/https/4.jpg)

- 功能：

> 将证书文件test.crt导入到名为test_cacerts的证书库中，

5.查看证书信息

```
keytool -printcert -file "test.crt"   
```

 ![img](/images/https/5.jpg)

- 功能：查看证书文件test.crt的信息

执行完上述过程之后，我们会在<JAVA_HOME>\bin目录下面发现生成了3个文件



### 修改web程序的web.xml

```
<security-constraint> 
		   <web-resource-collection> 
				  <web-resource-name>SSL</web-resource-name> 
				  <url-pattern>/*</url-pattern> 
		   </web-resource-collection>                           
		   <user-data-constraint> 
				  <transport-guarantee>CONFIDENTIAL</transport-guarantee> 
		   </user-data-constraint> 
	</security-constraint>
```

- url-pattern:可以对系统中访问的路径进行过滤

### 修改tomcat的conf文件

```
<Connector port="72" protocol="HTTP/1.1"    connectionTimeout="20000"
                redirectPort="8443" 
			   SSLEnabled="true"   maxThreads="150" 
			   scheme="https" secure="true"   clientAuth="false"
			   keystoreFile="C:/tomcat/conf/test.keystore"   
			   keystorePass="123456" sslProtocol="TLS"
			   URIEncoding="UTF-8"/>
```

- keystoreFile：证书文件的位置
- keystorePass: 是keystore的密码（你在生成证书的时候，会有的keystore密码和tomcat主密码）

### 

## 使用阿里云服务器生成免费证书

### 生成证书

基于阿里云服务器生成密钥证书，请参考[用阿里云的免费 SSL 证书让网站从 HTTP 换成 HTTPS](https://ninghao.net/blog/4449) 。一般如果证书申请时填写的信息正确，审核不超过10min，在审核通过之后我们选择对应的web服务器，不同的服务器的配置都不相同，这里我们使用ngnix服务器。（其他的web服务器可以在官文中查看，每种服务器都给出了基本的配置方法。）

> 阿里云免费证书只能对一个域名生效，如果涉及到二级域名或是其他域名，则需要申请不同的证书。

### 配置web服务器

1、 在Nginx的安装目录的conf下创建cert目录，并且将下载的全部文件拷贝到cert目录中。生成证书时，如果是系统自动生成会自己带有私钥文件，如果是自己创建的CSR文件，请将对应的私钥文件放到cert目录下并对其重命名建议和.pem证书文件的名称保持一致；

2、 打开 Nginx 安装目录下 conf 目录中的 nginx.conf 文件进行修改(以下属性中ssl开头的属性与证书配置有直接关系，其它属性请结合自己的实际情况复制或调整)：

```
server {
    listen 443;
    server_name abc.test.com;
    ssl on;
    root html;
    index index.html index.htm;
    ssl_certificate   cert/xxx.pem;
    ssl_certificate_key  cert/xxx.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        root html;
        index index.html index.htm;
    }
    location /test{
        tcp_nodelay     on;  
        proxy_set_header Host            $host;  
        proxy_set_header X-Real-IP       $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        proxy_pass http://www.baidu.com;  
    }
}
```

>- 1、 server_name 为系统监听的访问的服务域名。和阿里云中申请的证书的域名保持一致。
>- 2、ssl_certificate 为证书文件的路径。
>- 3、ssl_certificate_key 为私钥文件的路径。
>- 4、在location中路径结合实际情况进行设置。其中/test则是对https的请求进行了转发操作。如何涉及到二级域名的转发操作，则可以在server下面在增加一个server配置项进行相关的配置。

 3、启动 nginx，如果在启动时修改了配置文件则重启nginx即可。

 4、通过 https 方式访问您的站点，测试站点证书的安装配置。

>  使用nginx服务器我们不需要对系统的后台服务器进行其他操作。



## 其他相关文章

1、[SSL证书与Https应用部署小结](https://blog.csdn.net/andy1219111/article/details/22716315)

2、[https知识了解](https://blog.csdn.net/andy1219111/article/details/22716315)

