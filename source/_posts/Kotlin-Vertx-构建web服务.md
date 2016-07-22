---
title: Kotlin & Vertx 构建web服务
date: 2016-07-22 22:01:39
tags: Kotlin,Vertx
categories: Kotlin,Vertx
---

## 感想

>Kotlin 是一门好语言，值得大家了解一下。

>Vertx 是一个好框架，也值得大家了解一下。

#### Kotlin
写过js，也写过一点点go，主力一直是java。用了kotlin，貌似找到了常用语言的平衡点了。

Kotlin 拥有一些偏函数式的语法（java8 也引入了一些），提供了相当多便捷的api与一些高阶函数。从两天的试用，以及今天搞得这个 Vertx web 项目，从中体会到最爽的有两点：

* 支持“带接收者得函数字面值”(允许你直接指定函数的receiver的类型)这一特性。这个特性，在go里面经常看到。然而，java没有，java8也没有...  
* 支持扩展函数（或许是我见识短，这功能爆炸了）

一直很期待可以指定receiver这个功能。有了这个特性，那么写的函数，可以直接被调用者使用。

#### Vertx
vertx 风格和node的express框架思想一致的，换了一种java的实现。不得不说，node的express 启发了很多其他语言的web框架设计。java的vertx，以及go里面的很多web框架(martin...)，很多都有express的影子(难道是我先入为主？)

相比传统的基于Servlet的java web框架，vertx这种基于封装底层通信的框架，在速度上和内存占用上比较有优势。曾经为了在768M内存的docker容器上跑web应用，先是用相对较轻量级的spring-boot,勉强可以跑。然后又用了Node 的 express，这个毫无压力。

终于有一个java版本的这种web框架，整个项目打完包，包括依赖的 lib，整个才4-5M的大小（主要是lib大小），太轻量了。

下面就看看 Kotlin + Vertx 写的web项目，展示下kotlin的魅力。你要是比较懒的话，想直接check out代码，github库[在这里](https://github.com/ThinkCats/Vertx.git)。

### Kotlin & Vertx

#### 1.maven 配置
 按照kotlin和vertx官方的配置。
 ```xml
 <?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>vertx</groupId>
    <artifactId>com.vertx</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>com.tt.vertx.HelloWorlds</Main-Class>
                                        <Main-Verticle>com.tt.vertx.HelloWorlds</Main-Verticle>
                                    </manifestEntries>
                                </transformer>
                            </transformers>
                            <artifactSet/>
                            <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-fat.jar</outputFile>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.jetbrains.kotlin</groupId>
                <artifactId>kotlin-maven-plugin</artifactId>
                <version>${kotlin.version}</version>
                <executions>
                    <execution>
                        <id>compile</id>
                        <phase>process-sources</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <sourceDirs>
                                <source>src/main/java</source>
                            </sourceDirs>
                        </configuration>
                    </execution>
                    <execution>
                        <id>test-compile</id>
                        <phase>process-test-sources</phase>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                        <configuration>
                            <sourceDirs></sourceDirs>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-web</artifactId>
            <version>3.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
    </dependencies>
    <properties>
        <kotlin.version>1.0.3</kotlin.version>
    </properties>

</project>

 ```

#### 2.程序入口
和spring-boot 一样，vertx 的程序入口也是一个静态的main函数。

```java
class  HelloWorlds : AbstractVerticle() {

    companion object{
        @JvmStatic fun main(args: Array<String>){
            var vertx = Vertx.vertx()
            vertx.deployVerticle(HelloWorlds())
        }
    }

    override fun start() {
        var router = customRouter(vertx)
        println("server running on 8888")
        vertx.createHttpServer().requestHandler({ handler -> router.accept(handler)}).listen(8888)
    }
}
```
如果写过nodejs应用，这段代码看起来就很简单了。继承vertx的启动类，在启动的时候，设置路由，绑定http server的端口。

`注意`
> 和普通的kotlin运行类不一样，入口主类需要打jar包的时候，主方法（main）一定是要是java的标准静态方法. 需要加 @JvmStatic 标注一下。否则运行打出来的jar包，会找不到静态主入口。

在这里，封装了一下路由，提取到了单独的文件中。

#### 3.路由设置
```java
fun customRouter(vertx : Vertx) : Router {
    var router = Router.router(vertx)
    router.route("/").handler({c -> c.response().html().end("hello world")})
    router.route("/json").handler({c -> c.response().json().end(Json.encode(Entity("name","sss")))})
    return router
}

fun HttpServerResponse.html() : HttpServerResponse {
    return this.putHeader("content-type","text/html")
}

fun HttpServerResponse.json() : HttpServerResponse {
    return this.putHeader("content-type","application/json; charset=utf-8")
}
```

让我觉得kotlin拯救了我的地方就在这一段路由代码里。

先看一下，之前用纯java写的路由版本是什么样的吧，对比一下。

java版本：
```java
router.route("/").handler(context -> context.response().putHeader("content-type","text/html").end("hello world"));
  router.route("/json").handler(context -> {
      context.response().putHeader("content-type","application/json; charset=utf-8")
              .end(Json.encodePrettily(new Entity("hello","world")));
  });
```

在java版本里面，给每个请求加header的时候，是要加 `.putHeader("content-type","xxxx")` 的。

试想一下，如果我们有上百个路由的话，每个头里面都要指定这些header，多痛苦。根据经验，我们可能想把它提出来，当做一个方法。于是我们需要写一个工具类之类的，给`context.response()`的结果 HttpServerResponse 加上header，然后再返回这个 HttpServerResponse，最直观的大概写法应该是：
```
SomeUtil.setHeader(context.response()).end('xxx')
```
不管怎么说，因为java的语法限制，我们貌似只能这么写（如果有更好的方法，请大家共享一下）。

看一下kotlin版本的，把加header的操作提了出来，弄成了两个函数。这两个函数是函数接收字面量和扩展函数的结合体，使用扩展函数，这是个，也必须声明一个函数的接受者类型。
我们定义html() 和 json() 函数， 这两个方法只能用HttpServerResponse 去执行它，并且返回执行后的该 HttpServerResponse。 在扩展函数里面，this指向的是该函数的调用者。语法官方有详细文档，就少说了（好多语法我也没有看完 ORZ ）。

这个特性超级好用，尤其是在这种链式的调用里面，我们不用在去另写写转换方法，不用在把要转换的对象当参数来回传递了。

#### 4. 还有一个bean 对象
```java
data class Entity(var name:String,var description:String){}
```
第一次写这个bean的时候，我写错了，无论如何，在路由里面，就是无法取到该bean构造后的值。看了官方文档才知道，这个类是需要加一个data前缀的，这样才会生成 equals ，hashCode ，Setter, Getter 等这些方法。

### 另外一些感想
主要代码都在这里了，完整的代码[在这里](https://github.com/ThinkCats/Vertx.git)。

用kotlin , 感觉很幸福，相见恨晚。

代码写的少，对java 是一种强力的补充。它和java之间，完全可以相互调用，是一门很不错的语言，以后可以慢慢用起来了，还有很多东西值得去探索。

还有一点还是提一提吧，kotlin亲爹jetbrains, Idea 亲爹也是 jetbrains。Idea对自家的这门语言支持的相当的不错，大家都可以试试。另： 放弃Eclipse吧，别再折磨自己啦。

好久没写这么长的文章了，现在国内kotlin相关的东西也不多，希望这篇能有点作用。国内好多文字都随意转载，经常见到不署名原作作者地址的。知识靠传播才能影响更多人，也希望大家转载的话标注一下文章的原地址。
