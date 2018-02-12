---
Title: Spring Boot开发 web 应用 - 01 创建项目
Description: Spring Boot 开发web 应用 - 01 创建项目; 多种方式创建项目
---
Spring Boot非常适合Web应用程序开发。使用spring-boot-starter-web模块快速启动和运行。 其中使用嵌入式Tomcat，Jetty或Undertow轻松创建自包含的HTTP服务器

Spring Boot支持多种方式来创建一个项目：

 1. 使用curl命令 [Using curl](https://spring.io/guides/tutorials/spring-security-and-angular-js/#using-curl)
 2. 使用Spring Boot CLI 命令行工具 [Using CLI](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-installing-the-cli)
 3. https://spring.io/  在线生成项目
 4. 使用Spring的集成开发工具

## 创建Spring Boot项目：
这里简单在线生成一个项目; 直接访问https://start.spring.io/，填写Group和Artifact，添加Web和 Freemarker两个依赖项。
![这里写图片描述](http://img.blog.csdn.net/20170613085631210?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hvZWxlYQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> Spring Boot 支持的可以替换JSP的，view engine包括： Thymeleaf, Groovy Markup Templates, Freemarker, Velocity. (As of Spring Framework 4.3, Velocity support has been deprecated due to six years without active maintenance of the Apache Velocity project. ) 除了Velocity 以外，其他根据个人爱好自助选择。

## 添加HomeController
```
@Controller
public class HomePageController {
	
	@RequestMapping("/")
	public String home(){
		return "index";
	}
}
```

## 添加前端页面的模板
采用开源bootstrap的H5 模板：[awesome template](http://www.templatemo.com/preview/templatemo_450_awesome)
默认页面文件需要添加至/src/main/resources/templates 下面；静态资源文件的位置在/src/main/resources/static (或者/src/main/resources/public)下面。
目录结构如下：
![这里写图片描述](http://img.blog.csdn.net/20170613091118932?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hvZWxlYQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

运行SpringBootApplication主程序；访问http://localhost:8080验证。 截止目前，没有任何定制的情况下，我们可以看到Awesome 的模板页面。

## github repo
[spring-boot-trail-static-content](https://github.com/choelea/spring-boot-trail-static-content.git)   tags/ootb