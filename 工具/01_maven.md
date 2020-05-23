# 概述
maven是专门为java项目打造的管理金和构建工具，主要功能有：
- 提供一套标准化的项目结构
- 提供一套标准化的构建流程（编译、测试、打包、发布...）
- 提供一套依赖管理机制
 maven项目结构：
 ```
mavenProject
|
|-- pom.xml //项目描述文件
|-- src
|   |
|   |--main
|   |  |
|   |  |-- java     //存放源码
|   |  |-- resources    //存放资源
|   |
|   |-- test        //存放测试代码
|   
|-- target      //存放生成文件
 ```

 pom.xml:
 ```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.housq</groupId>    //公司或组织名
    <artifactId>mavenTest01</artifactId>    //项目名称
    <version>1.0-SNAPSHOT</version>     //版本号

    <packaging>jar</packaging>      //打包类型

    <dependencies>     //声明依赖

        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api --> //依赖项，通过groupId、artifactId、version唯一确定一个依赖库
        <dependency> 
            <scope>compile</scope>      //依赖范围
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>

    </dependencies>
</project>
 ```

 # 依赖管理
 只需声明项目的依赖，maven会自动导入依赖包，以及依赖包的依赖包。
 maven定义了几种依赖关系：
 - compile：编译时需要拥到的jar包（默认），如：commons-logging
 - test：编译test时需要用到的jar包，如：junit
 - runtime：编译时不需要，但运行时需要，如mysql
 - provided：编译时需要用到，运行时由jdk或某个服务器提供，如：servlet-api

 maven维护了一个[中央仓库](https://repo1.maven.org/),所有第三方仓库将自身的jar以及相关信息上传至中央仓库，maven就可以从中央仓库把所需依赖下载到本地，并缓存。

 maven通过groupId、artifactId和version唯一确定某个jar包，maven通过对jar包进行PGP签名确保任何一个jar包一经发布就无法修改，修改的唯一方法就是发布一个新版本。

 除了从maven中央仓库下载外，还能从maven的镜像仓库下载，maven镜像仓库定期从中央仓库同步。使用镜像仓库需要配置：在用户主目录下进入.m2目录，创建settings.xml文件：
 ```
<settings>
    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>aliyun</name>
            <mirrorOf>central</mirrorOf>
            <!-- 国内推荐阿里云的Maven镜像 -->
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </mirror>
    </mirrors>
</settings>
 ```

查找一个第三方组件可以通过<https://mvnrepository.com/>搜索关键字，找到之后直接复制即可。

# 构建流程
maven有一套标准化的构建流程，可以实现编译、打包、发布等。

## Lifecycle和Phase
maven的声明周期由一系列阶段(phase)构成，以内置的生命周期default为例，包含的phase有：
- validate
- initialize
- generate-sources
- process-source
- generate-resources
- process-resources
- compile
- process-classes
- process-classes
- generate-test-sources
- process-test-sources
- generate-test-resources
- process-test-resources
- test-compile
- process-test-classes
- test
- prepare-package
- package
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install
- deploy

如果运行```mvn package```，maven就会执行default声明周期，会从开始一直运行到package这个phase为止。

另一个常用的声明周期clean，包含3个phase：
- preclean
- clean
- post-clean

所以执行mvn命令时，后面的参数叔phase，maven自动根据生命周期运行到指定的phase。更复杂的情况是知道多个phase，如：```mvn clean package```，maven先执行clean声明周期到clean，然后执行default声明周期到package，经常用到的命令有：
- mvn clean
- mvn clean compile
- mvn clean test
- mvn clean package

大多数phase执行过程中，因为通常没有在pom.xml中配置相关的设置，所以这些phase什么都不做。

执行一个phase又会触发一个或多个goal：
- compile -- compiler：compiler
- test -- compiler：testCompiler/surefile:test

goal的命名总是abc:xyz这种形式。

# 插件
mvn执行每个phase，都是通过某个插件（plugin）来执行的，maven只负责找到对应的插件，然后执行对应的对应的goal。maven已经内置了一些常用的标准插件：
|插件名称| 对应的phase |
|:-- | :--|
| clean | clean |
| compiler | compile |
| surefire | test |
| jar | package |
如果标准插件无法满足需求，还可以使用自定义插件。使用自定义插件需要在pom.xml中声明(maven-shade-plugin)：
```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>com.itranswarp.learnjava.Main</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

常用的插件：
- maven-shade-plugin：打包所有依赖包，并生成可执行的jar
- cobertura-maven-plugin：生成单元测试覆盖率报告
- findbugs-maven-plugin：对java代码进行静态分析以找出潜在问题

# 模块管理
maven可以有效的管理多个模块，同时也是开发大项目时降低复杂度的好方法。每个模块当做独立的maven项目，有各自独立的pom.xml。同时可以把多个模块共同的部分分离出来作为parent,parent的packaging是pom。
此时工程目录：
```
mavenProject
|
|-- pom.xml
|
|-- parent
|   |
|   |-- pom.xml
|
|-- midule-a
|   |
|   |-- pom.xml
|
|-- midule-b
|   |
|   |-- pom.xml
```

parent：
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>

    <name>parent</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.28</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.5.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

module-a的pom就可以简化为：
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.itranswarp.learnjava</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
        <relativePath>../parent/pom.xml</relativePath>
    </parent>

    <artifactId>module-a</artifactId>
    <packaging>jar</packaging>
    <name>module-a</name>
</project>
```

如果module-a依赖于module-b，需要在module-a中引入module-b：
```
<dependencies>
    <dependency>
        <groupId>com.itranswarp.learnjava</groupId>
        <artifactId>module-b</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```
最后在编译的时候，需要在跟目录创建一个pom.xml统一编译：
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>build</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>
    <name>build</name>

    <modules>
        <module>parent</module>
        <module>module-a</module>
        <module>module-b</module>
        <module>module-c</module>
    </modules>
</project>
```

# mvnW
mvnw是Maven Wrapper的简写，因为在安装时，默认情况下所有项目都会使用全局安装的这个maven版本。但是对于某些项目可能需要特定的版本才行，这个时候可以使用maven wrapper，他可以负责给这个特定项目制定版本的maven。

安装制定版本的maven，可在pom.xml所在的目录，运行安装命令：
```mvn -N io.takari:maven:0.7.6:wrapper -Dmaven=3.6.3```
其中0.7.6是maven wrapper的版本，最新的版本可以去[官网](https://github.com/takari/maven-wrapper)查看。3.6.3是maven版本。

之后只需要把mvn命令换成mvnw即可。

mvnw的另一个作用是把项目的mvnw、mvnw.cmd和.mvn提交到版本库中，所以的开发人员使用统一的maven版本。

# 发布Artifact
