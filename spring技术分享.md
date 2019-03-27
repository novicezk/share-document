title: Spring 技术分享
speaker: 朱凯
plugins:
    - echarts

<slide class="bg-black-blue aligncenter">

# Spring 技术分享 {.text}
## 朱凯 {.text-intro.animated}


<slide class="bg-black-blue aligncenter">

## Spring 系列常用框架简介

<slide class="bg-black-blue" image="./spring-project.jpeg">

1．Spring Core
Core模块是Spring的核心类库，Spring的所有功能都依赖于该类库，Core主要实现IOC功能，Sprign的所有功能都是借助IOC实现的。
2．AOP
AOP模块是Spring的AOP库，提供了AOP（拦截器）机制，并提供常用的拦截器，供用户自定义和配置。
3．ORM
Spring 的ORM模块提供对常用的ORM框架的管理和辅助支持，Spring支持常用的Hibernate，ibtas，jdao等框架的支持，Spring本身并不对ORM进行实现，仅对常见的ORM框架进行封装，并对其进行管理。
4．DAO模块
Spring 提供对JDBC的支持，对JDBC进行封装，允许JDBC使用Spring资源，并能统一管理JDBC事物，并不对JDBC进行实现。
5．WEB模块
WEB模块提供对常见框架如Struts1，WEBWORK（Struts 2），JSF的支持，Spring能够管理这些框架，将Spring的资源注入给框架，也能在这些框架的前后插入拦截器。
6．Context模块
Context模块提供框架式的Bean访问方式，其他程序可以通过Context访问Spring的Bean资源，相当于资源注入。
7．MVC模块
WEB MVC模块为Spring提供了一套轻量级的MVC实现，在Spring的开发中，我们既可以用Struts也可以用Spring自己的MVC框架，相对于Struts，Spring自己的MVC框架更加简洁和方便。

<slide>
!![](https://novicezk.github.io/spring-model.jpg .size-50.alignleft)

<!-- Spring 技术分享
一、 Spring 系列常用框架简介
二、 Spring Framework
    1. DI
    2. AOP
    3. MVC
三、Spring Data
四、Spring Boot -->