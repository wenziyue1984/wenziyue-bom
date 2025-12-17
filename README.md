# **wenziyue-bom**





这是一个 **Maven BOM（Bill of Materials）** 项目，用来做 **统一版本管理**。它本身不提供业务代码，只提供一份可复用的 dependencyManagement，把你的 **starter / core / 各个服务** 在三方依赖上拉到同一条版本线上，减少“同一个 jar 在不同模块被拉出不同版本”导致的运行期问题。



------





## **这个 BOM 解决什么问题**





在你的生态里，存在三类模块：



1. **基础库 / 中间模块**：比如 wenziyue-*-starter、wenziyue-auth-core

   它们会被多个服务引用，如果它们各自写死版本或版本漂移，就容易出现依赖冲突。

2. **终点服务**：比如 blog-gateway-service、blog-user-service

   它们组合多个 starter / core，更容易把依赖版本拉乱。

3. **Spring Boot / Spring Cloud**：它们自己也带了一套依赖版本基线

   只要你统一了 Boot/Cloud 的版本，很多三方库版本就天然一致。





所以 BOM 的目标是：让所有模块 **共享同一套版本决策**，并且让你升级版本时只改一个地方。



------





## **BOM 管哪些版本**





这个 BOM 里做了两件事：



- 引入 **Spring Boot BOM** 与 **Spring Cloud BOM**，作为生态基线（Boot/Cloud 自己会管理大量依赖版本）。
- 对 **Boot/Cloud 没有管理** 或者你希望显式锁定的三方库做版本管理，例如：rocketmq-spring-boot-starter、guava、hutool、springdoc、jjwt 等。





同时它也管理自己生态的版本，例如：wenziyue-redis-starter、wenziyue-auth-core 等。



------





## **怎么在项目里使用**





你有两种用法，推荐第一种。





### **用法 1：在dependencyManagement里 import BOM（推荐）**

在你的服务或 starter 的 pom.xml 里加入：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.wenziyue</groupId>
            <artifactId>wenziyue-bom</artifactId>
            <version>1.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后你在 <dependencies> 里引入依赖时，**尽量不写版本号**，交给 BOM（或 Boot BOM）来决定：

```xml
<dependencies>
    <dependency>
        <groupId>com.wenziyue</groupId>
        <artifactId>wenziyue-redis-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.apache.rocketmq</groupId>
        <artifactId>rocketmq-spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```



### **用法 2：让项目继承一个 parent（可选）**



如果你后面还想统一插件版本、编码、编译参数、仓库发布配置，可以再建一个 **parent** 项目；但 BOM 只负责版本对齐，保持干净即可。



------





## **一个重要原则：BOM 不要反向依赖 starter**



**BOM 可以管理 starter 的版本，但 starter 不要依赖 BOM 来管理“自己”的版本。**



原因很简单：

如果 BOM -> starter版本，同时 starter -> BOM，会形成版本发布上的循环依赖。



------





## **如何判断某个依赖是否已经被 Spring Boot 管理版本**





你可以用这几个方法：



1. **最直接**：在项目里先不写 <version>，如果 Maven 构建不报 “version is missing”，通常说明版本已被上游 BOM 管理。
2. 看有效 POM（effective-pom）或依赖树（dependency tree）：
    - mvn -DskipTests help:effective-pom
    - mvn -DskipTests dependency:tree



如果你之前执行 mvn -q -DskipTests help:effective-pom 没输出，是因为 -q 把输出压没了。把 -q 去掉，或者重定向到文件：

```shell
mvn -DskipTests help:effective-pom > effective-pom.xml
```



------





## **版本升级建议**



升级时尽量按顺序做：



1. 先选定 **Spring Boot / Spring Cloud** 的配对版本（保持兼容）。
2. 再升级 BOM 里显式锁定的三方依赖（rocketmq、springdoc、guava 等）。
3. 最后升级你的 starter/core 版本（如果你有“stack-bom”来统一，会更舒服）。





------



