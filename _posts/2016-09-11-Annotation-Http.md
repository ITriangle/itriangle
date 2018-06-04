---
layout: post
title: HTTP 请求参数绑定的注解
categories: Annotation
description: Annotation 是 Spring Boot 的一种配置
keywords: SpringBoot,Annotation,Java
---

## 参数绑定简介:
handler method 参数绑定常用的注解,根据他们处理的Request的不同内容部分分为四类：

1. 处理 `requet uri` 部分(这里指uri template中variable，不含queryString部分)的注解：`@PathVariable`;
2. 处理 `request header` 部分的注解: `@RequestHeader`, `@CookieValue`;
3. 处理 `request body` 部分的注解: `@RequestParam`,  `@RequestBody`;
4. 处理 `attribute` 类型是注解: `@SessionAttributes`, `@ModelAttribute`;

### 1. @PathVariable  

当使用 `@RequestMapping` 样式映射时， 即 `someUrl/{paramId}`, 这时的`paramId`可通过 `@Pathvariable` 注解绑定请求中的值,传递到方法的参数上。

```java
@Controller  
@RequestMapping(“/owners/{ownerId}”)  
public class RelativePathUriTemplateController {      
  @RequestMapping(“/pets/{petId}”)  
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {      
    // implementation omitted   
  }  
}  
```

上面代码把`URI template`中变量`ownerId`的值和`petId`的值，绑定到方法的参数上。**若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable(“name”)指定uri template中的名称。**  


### 2. @RequestHeader、@CookieValue  

#### @RequestHeader 
注解，可以把Request请求`header部分`的值绑定到方法的参数上。

如下是Request 的`header部分`：

```
Host                     localhost:8080  
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9  
Accept-Language   fr,en-gb;q=0.7,en;q=0.3  
Accept-Encoding    gzip,deflate  
Accept-Charset      ISO-8859-1,utf-8;q=0.7,*;q=0.7  
Keep-Alive             300  
```

```java
@RequestMapping(“/displayHeaderInfo.do”)  
public void displayHeaderInfo(@RequestHeader(“Accept-Encoding”) String encoding,  @RequestHeader(“Keep-Alive”) long keepAlive)  {  
}  
```

上面的代码，把request header部分的 `Accept-Encoding`的值，绑定到参数`encoding`上了， `Keep-Alive header`的值绑定到参数`keepAlive`上。

#### @CookieValue 
可以把Request header中关于 `cookie`的值绑定到方法的参数上。例如有如下Cookie值：`JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84 `

```java
@RequestMapping(“/displayHeaderInfo.do”)  
public void displayHeaderInfo(@CookieValue(“JSESSIONID”) String cookie) {  
  ...      
}  
```

即把JSESSIONID的值绑定到参数cookie上。

### 3.@RequestParam, @RequestBody  

#### @RequestParam  

1. 常用来处理简单类型的绑定，通过`Request.getParameter()` 获取的`String`可直接转换为简单类型的情况（ String –> 简单类型的转换操作由`ConversionService`配置的转换器来完成）；因为使用`request.getParameter()`方式获取参数，所以可以处理`get` 方式中`queryString`的值，也可以处理`post`方式中 `body data`的值；
2. 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST；
3. 该注解有两个属性: value、required; value用来指定要传入值的id名称,required用来指示参数是否必须绑定；

```java
@Controller  
@RequestMapping(“/pets”)  
@SessionAttributes(“pet”)  
public class EditPetForm {  
    @RequestMapping(method = RequestMethod.GET)  
    public String setupForm(@RequestParam(“petId”) int petId, ModelMap model) {  
       Pet pet = this.clinic.loadPet(petId);  
       model.addAttribute(“pet”, pet);  
       return “petForm”;  
 } 
```


#### @RequestBody
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容，处理完的结果放在一个MultiValueMap里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;

```java
@RequestMapping(value = “/something”, method = RequestMethod.PUT)  
public void handle(@RequestBody String body, Writer writer) throws IOException {  
  writer.write(body);  
} 
```

### 4.@SessionAttributes, @ModelAttribute

#### 　@SessionAttributes
该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象；

```java
@Controller  
@RequestMapping(“/editPet.do”)  
@SessionAttributes(“pet”)  
public class EditPetForm {  
        // …   
 } 
```

以上代码:结合`@RequestParam`实例查看,通过字符串直接匹配对象的值

#### @ModelAttribute 
该注解有两个用法，一个是用于方法上，一个是用于参数上；
1. 用于方法上时：通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；

```java
// Add one attribute   
// The return value of the method is added to the model under the name “account”   
// You can customize the name via @ModelAttribute(“myAccount”) 
@ModelAttribute  
public Account addAccount(@RequestParam String number) {  
    return accountManager.findAccount(number);  
} 
```

这种方式实际的效果就是在调用@RequestMapping的方法之前，为request对象的model里put（“account”， Account）；

2. 用于参数上时：用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：
    - @SessionAttributes启用的attribute 对象上；
    - @ModelAttribute用于方法上时指定的model对象；
    - 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中。

```java
@RequestMapping(value=“/owners/{ownerId}/pets/{petId}/edit”, method = RequestMethod.POST)  
public String processSubmit(@ModelAttribute Pet pet) {  
    ...
}   
```

## 处理流程
首先查询 `@SessionAttributes` 有无绑定的Pet对象，若没有则查询 `@ModelAttribute`方法层面上是否绑定了Pet对象，若没有则将`@PathVariable `中的值按对应的名称绑定到Pet对象的各属性上。

## 参考链接
1. [@PathVariable和@RequestParam的区别,@SessionAttributes](http://www.blogs8.cn/posts/A1mIcd8)