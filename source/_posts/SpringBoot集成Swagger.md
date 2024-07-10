---
title: SpringBoot集成Swagger
author: 荀乐
tags:
 - Swagger
 - SpringBoot
 - Java
date: 2021-07-06 13:39:26

---

# SpringBoot集成Swagger



这是我参与新手入门的第一篇文章

---

## 前言

在Web后端人员开发接口时，需要对接口进行测试，这是我们可以使用到今天要提到的工具Swagger。



## Swagger是什么？

-   Swagger官网网址：[API Documentation & Design Tools for Teams | Swagger](https://swagger.io/)

进入官网我们就可以看见官方对Swagger的简介

>   Simplify API development for users, teams, and enterprises with the Swagger open source and professional toolset. Find out how Swagger can help you design and document your APIs at scale.

简言之就是帮助用户简化API开发的工具，它可以将API可视化从而方便开发人员进行设计于调试。

由于笔者主要使用Java，故今天想要介绍的是在SpringBoot中如何集成Swagger。



## 新建SpringBoot项目并引入依赖

-   将Swagger的依赖引入SpringBoot项目

    ```xml
    <!-- https://mvnrepository.com/artifact/io.springfox/springfox-boot-starter -->
    <dependency>
    	<groupId>io.springfox</groupId>
    	<artifactId>springfox-boot-starter</artifactId>
    	<version>3.0.0</version>
    </dependency>
    ```

-   项目结构如图

-   ![image-20210706155932304](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206154111.png)

    



## 新建Swagger配置类

Swagger配置类位置如上图

Swagger配置类用于配置Swagger扫描的接口、作者的信息

配置类代码如下

```java
/**
 * @author xunle
 */
@EnableSwagger2
@Configuration
public class SwaggerConfig {
    /**
     * 配置Swagger的信息
     */
    private ApiInfo getApiInfo() {
        //作者信息
        Contact contact = new Contact("xunle","https://xunle1.github.io/","1601315809@qq.com");
        return new ApiInfoBuilder()
                //标题
                .title("Swagger Test")
                //描述信息
                .description("SpringBoot集成Swagger")
                //版本号
                .version("v1.0")
                //作者信息
                .contact(contact)
                .build();
    }

    /**
     * 用于配置Swagger扫描接口
     */
    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(getApiInfo())
                .select()
                //apis用于配置扫描哪些接口
                .apis(RequestHandlerSelectors.basePackage("com.example.swagger.controller"))
                //paths用于配置路径选择
                .paths(PathSelectors.ant("/hello/**"))
                .build()
                //是否开启Swagger
                .enable(true);
    }
}
```



## 新建Controller

Controller位置如上图

Controller类中定义接口

```java
/**
 * @author xunle
 */
//Api 类信息
@Api(tags = "Swagger测试Controller")
@Controller
public class SwaggerController {
    //ApiOperation 方法描述
    @ApiOperation("Swagger测试Get方法")
    @ResponseBody
    @GetMapping("/hello/test")
    public String test() {
        return "Swagger测试";
    }
    
    @ApiOperation("Swagger是否扫描路径")
    @ResponseBody
    @GetMapping("/hi/test")
    public String testPathSelector() {
        return "selected!";
    }
}
```



## Swagger-ui基本信息

一般情况下访问 ==工程名/swagger-ui.html==

笔者本机访问 http://localhost:8080/swagger-ui.html

![image-20210706162908770](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206154131.png)

前两步的配置都在Swagger-ui中显示出来了，可以对照着看配置对应的显示

注意Controller中配置的Path，第二个Get方法路径==/hi/test==不匹配Swagger配置类中的路径==/hello/**==，故不会显示在Swagger-ui中

## 接口测试

在点击对应API后

![image-20210706163636565](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206154136.png)

>   -   Parameters为参数列表
>   -   Response为返回信息
>       -   Code：状态码
>       -   Description：访问成功时Example Value中显示数据类型，Model表示返回的是实体类

点击Try it out后显示如下：

![image-20210706164104575](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206154139.png)

点击Execute进行接口测试后显示如下：

![image-20210706164349701](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206154143.png)

>   -   Response body：返回信息
>   -   Response headers：响应头
