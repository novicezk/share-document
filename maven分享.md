title: maven分享
speaker: 朱凯
url: https://github.com/novicezk
transition: cards
files: /js/demo.js,/css/demo.css

[slide]

# Maven 技术分享
## 分享者：朱凯

[slide]

# 一、Maven 是什么？
Maven是 Apache 下基于项目对象模型（POM）的项目管理工具，可以对 Java 项目进行构建、依赖管理。也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。

[slide]
# 二、Maven 环境配置

1. [配置java环境](http://www.runoob.com/java/java-environment-setup.html)
2. 下载 [maven安装包](http://maven.apache.org/download.cgi)
```bash
wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar -xvf  apache-maven-3.3.9-bin.tar.gz
sudo mv -f apache-maven-3.3.9 /usr/local/
```
3. 配置环境变量 `sudo vim /etc/profile` ，末尾添加
```bash
export MAVEN_HOME=/usr/local/apache-maven-3.3.9
export PATH=${PATH}:${MAVEN_HOME}/bin
```
4. 执行 `source /etc/profile` 使配置生效
5. mvn -v 检测是否配置成功

[slide]
# 三、Maven 配置文件

|类型|位置|
|--|--|
|全局|%M2_HOME%/conf/settings.xml|
|用户级|%USER_HOME%/.m2/settings.xml|
~/.m2/settings.xml 初始是没有的，需要自己创建，创建后在文件中配置的值会覆盖全局配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- 本地仓库路径，默认路径为${user.home}/.m2/repository -->
  <localRepository>/data/work/repository</localRepository>
  <!-- 离线模式，默认值为false。当因为各种原因无法与远程仓库连接时，可将该值修改为true(此时不会从镜像仓库更新依赖) -->
  <offline>false</offline>
  <!-- 插件组，内部由n个pluginGroup节点组成，默认自带了org.apache.maven.plugins和org.codehaus.mojo。若有特殊的需求是可进行扩展 -->
  <pluginGroups></pluginGroups>
  <!-- 网络代理设置 -->
  <proxies></proxies>
  <servers></servers>
  <!-- 镜像仓库，用来代替中央仓库（Maven默认的远程仓库） -->
  <mirrors>
    <mirror>
      <id>homolo</id>
      <mirrorOf>central</mirrorOf>
      <name>Homolo maven Mirror.</name>
      <url>http://nexus.homolo.org/repository/maven-public</url>
    </mirror>
  </mirrors>
  <!-- 用于覆盖在一个POM或者profiles.xml文件中的任何相同id的profiles节点 -->
  <profiles>
   <profile>
      <id>snapshot-repo</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>homolo-snapshot</id>
          <name>homolo snapshots</name>
          <url>http://nexus.homolo.org/repository/maven-snapshots</url>
          <layout>default</layout>
          <releases>
            <enabled>false</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
```

[slide]
# 四、使用原型创建项目
1. 下载原型项目
```bash
git clone git@git.homolo.org:archetype/homolo-webapp-archetype.git
```
2. 进入该项目目录，执行 `mvn install`
3. 到workspace目录, 执行 `mvn archetype:generate`
```bash
274: remote -> com.homolo.archetype:homolo-module-archetype (homolo mongo module project archetype.This archetype is for mongo's module projects.这个模板适用于mongodb数据库。)
275: remote -> com.homolo.archetype:homolo-webapp-archetype (-)
...
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 
# 选择根据哪个模板来创建项目，这里示例创建公司项目
275
Choose com.homolo.archetype:homolo-webapp-archetype version:
1: 3.0
2: 4.0-SNAPSHOT
Choose a number:
# 若该模板有多个版本，需要选择，创建新框架项目选择4.0
2
# 后续设置groupId，artifactId等
# 公司项目初始需要执行 sh init.sh
```

[slide]
# 五、maven项目结构

```
src
--main
----java(项目的java源代码)
----resources(项目的资源,配置文件等)
--test
----java(项目的测试类)
----resources(测试用的资源)
target
--classes(编译输出目录)
--generated-sources
--test-classes(测试编译输出目录)
pom.xml(描述项目如何构建、声明项目依赖等)
```

[slide]
# 六、pom.xml 常用配置项
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.lenchy.apps</groupId>
    <artifactId>mediation-learn-webapp</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    <!-- 指定继承的父pom，会具有父pom已有的配置项 -->
    <parent>
        <groupId>com.homolo.framework</groupId>
        <artifactId>homolo-boot</artifactId>
        <version>5.0.0-SNAPSHOT</version>
    </parent>
    <!-- 以值替代名称，Properties可以在整个POM中使用 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <framework.version>4.1</framework.version>
    </properties>
    <build>
        <!--产生的文件名，默认值是${artifactId}-${version}。 -->
        <finalName>mediation-learn-webapp</finalName>
        <!-- 描述了项目相关的所有资源路径列表 -->
        <resources>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
            </resource>
            <resource>
                <directory>${basedir}/src/main/jsvm2</directory>
            </resource>
        </resources>
        <!-- build使用的插件列表 -->
        <plugins>
        </plugins>
    </build>
    <!-- 描述了项目相关的所有依赖 -->
    <dependencies>
        <dependency>
            <groupId>com.homolo.framework</groupId>
            <artifactId>homolo-framework</artifactId>
            <version>${framework.version}</version>
        </dependency>
    </dependencies>
     <!-- 继承自该项目的所有子项目的默认依赖信息 -->
    <dependencyManagement>
    <dependencies></dependencies>
    </dependencyManagement>
    <!-- 代码版本库信息 -->
    <scm>
        <connection>scm:git:git@git.homolo.org:lawyer/mediation-learn-webapp.git</connection>
        <developerConnection>scm:git:git@git.homolo.org:lawyer/mediation-learn-webapp.git</developerConnection>
        <url>https://git.homolo.org/lawyer/mediation-learn-webapp</url>
        <tag>HEAD</tag>
    </scm>
</project>
```

[slide]
# 七、pom 插件
Maven 是一个依赖插件执行的框架，每个任务实际上是由插件完成。例如编译源代码 `mvn compiler` 是由maven-compiler-plugin完成的。

----

Maven 插件通常被用来：
* 创建 jar 文件
* 创建 war 文件
* 编译代码文件
* 代码单元测试
* 创建工程文档
* 创建工程报告

[slide]
# 常用插件

[slide]

## compiler
项目编译时，指定编译器并设定参数，可以指定项目源码的jdk版本，编译后的jdk版本，以及编码等
```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.8</source> <!-- 源代码使用的JDK版本 -->
        <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->             
        <!-- 字符集编码 -->                                                         
        <encoding>UTF-8</encoding>
    </configuration>
</plugin>
```
[slide]

## surefile
用于Maven项目的test阶段，以执行单元测试
```xml
      <plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>2.19.1</version>
				<configuration>
          <!-- 可以设置排除规则，把一些不需要在test阶段执行的测试类排除 -->
					<excludes>
						<exclude>**/AppTest.java</exclude>
					</excludes>
				</configuration>
			</plugin>
```
[slide]

## war
用于将项目打成war包
```xml
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>2.5</version>
            <configuration>
                 <!-- 设置编码 -->
                 <encoding>UTF-8</encoding>
                 <!-- 设置名称 -->
                 <warName>test</warName>
            </configuration>
         </plugin>
```

[slide]
## findbugs
代码静态分析，可以用它来检查源代码中可能出现的问题
```xml
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>findbugs-maven-plugin</artifactId>
                <version>3.0.4</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>check</goal>
                        </goals>
                        <phase>compile</phase>
                    </execution>
                </executions>
                <configuration>
                    <xmlOutput>true</xmlOutput>
                    <threshold>Low</threshold>
                    <effort>Max</effort>
                    <failOnError>true</failOnError>
                    <findbugsXmlOutput>true</findbugsXmlOutput>
                </configuration>
            </plugin>
```
[slide]
## checkstyle
检查代码是否符合指定的编程规范
```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>2.17</version>
                <dependencies>
                    <dependency>
                        <groupId>com.puppycrawl.tools</groupId>
                        <artifactId>checkstyle</artifactId>
                        <version>7.6</version>
                    </dependency>
                </dependencies>
                <executions>
                    <execution>
                        <id>check</id>
                        <phase>process-test-sources</phase>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <configLocation>checkstyle.xml</configLocation>
                    <failOnViolation>true</failOnViolation>
                    <failsOnError>true</failsOnError>
                    <logViolationsToConsole>true</logViolationsToConsole>
                    <includeTestSourceDirectory>true</includeTestSourceDirectory>
                </configuration>
            </plugin>
```

[slide]
# 八、常见问题

[slide]
## 1. 两个依赖引用了不同版本的jar
[slide]

### maven 提供了多种防止依赖冲突的方法，常用的有以下几种：

1. 上级项目的pom在引用依赖时，可将<optional>设置为true, 其他项目引用该项目时，不会再自动引入optional标记的依赖，需要显式引用，例：
```xml
        <dependency>
            <groupId>org.json</groupId>
            <artifactId>json</artifactId>
            <version>20170516</version>
            <optional>true</optional>
        </dependency>
```
2. 上级项目设置依赖范围scope
  * compile ：默认范围，在编译和打包时都会将依赖存储进去 
  * provided：编译范围有效，在打包时不会将依赖存储进去
  * test    ：测试范围有效，在编译和打包时都不会使用这个依赖
  * runtime ：在运行的时候依赖，在编译的时候不依赖
3. 下游项目引用时可以设置exclusions，排除引用的上级项目中的部分依赖，例：
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

[slide]
## 2. 添加了依赖，更新pom后下载不了
[slide]
### 常见的有以下几种原因：
----

* maven设置为离线模式
* maven的setting.xml配错或少配了镜像仓库地址
* IDE设置的maven路径有误
* 本地repository已存在错误的该依赖目录，可删除后再次更新
* 镜像仓库中确实不存在该依赖

[slide]
## 3. 如何引入外部依赖(maven仓库之外的文件)

[slide]

* 把jar包传到本地或镜像仓库，然后正常引用
```bash
mvn deploy:deploy-file 
-DartifactId=dataexchange 
-DgroupId=com.thunisoft.summer
-Dversion=1.0.9 
-DrepositoryId=homolo 
-Durl=http://nexus.homolo.org/repository/thirdparty 
-Dfile=/home/rory/Desktop/dataexchange-1.0.9.jar
```
* 把jar包放到项目中，设置依赖范围为system，并设置systemPath
```xml
      <dependency>
            <groupId>com.test.project</groupId>
            <artifactId>test</artifactId>
            <version>1.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/libs/test.jar</systemPath>
      </dependency>
```

[slide]
# 谢谢大家