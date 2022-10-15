---
title: Spring Boot应用程序使用@valid来验证API请求
date: 2022-07-04 21:02:37
categories:
- SpringBoot
tags:
- Spring
- API
---

使用@valid可以使你的Request验证更容易。

{% asset_img 1.jpeg 示意图 width="400" %}

当我们开发rest API时，我们认为每个人都需要验证接口输入参数的合法性。
一些初学者可能会使用许多if-else条件表达式进行验证。以一个新的用户注册为例。

<!--more-->

```java
@RestController
public class DemoController {

    @PostMapping("/user/register")
    public Boolean registerUser(@RequestBody UserRegisterRequest request) {

        if (StringUtils.isBlank(request.getName() == null)) {
            throw new IllegalArgumentException("username is required");
        }

        if (StringUtils.isBlank(request.getpassword())) {
            throw new IllegalArgumentException("password is required");
        }

        if (StringUtils.isBlank(request.getEmail())) {
            throw new IllegalArgumentException("email is required");
        }
        
		..........
    }
}
```

这样的代码可以达到参数验证的目的，但是细心的读者可能已经发现，如果我们想写很多这样的接口，就需要重复写很多验证代码，这显然不优雅。
我们可以使用@valid注解来帮助我们简化验证逻辑。

#### 使用@Valid
hibernate-validator实现了@Valid。spring-boot-start-web已经包含这个库，所以不需要引入依赖关系。如果是非spring项目，需要使用两个Maven依赖项。

```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
 
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.14.Final</version>
</dependency>
```

以下是与@valid相关的注解。在实体类的属性中添加不同的注解可以实现验证功能。

 * @Null限制属性只能为空。
 * @NotNull限制属性不能为空。
 * @Max(value) 限制属性必须是一个不大于指定值的数字。
 * @Min(value)限制属性必须是一个不小于指定值的数字。
 * @Digits(integer, fraction) 限制属性必须是一个小数，而且整数部分的数字不能超过整数，小数部分的数字不能超过分数。
 * @过去限制属性必须是一个过去的日期。
 * @Future 限制属性必须是一个未来的日期。
 * @Pattern(value) 限制属性必须符合指定的正则表达式。
 * @Size(max, min) 限制字符长度必须在min和Max之间。
 * @NotEmpty有效属性值不能为空，字符串长度不能为0，集合大小不能为0。
 * @NotBlank 有效的属性值不是空的，并且在trim()之后，长度不是0。
 * @Email 验证属性值是一个电子邮件格式的字符串。

使用@valid注解来完成参数验证是非常简单的。你只需要在实体类的属性中添加适当的注解。
上面是同一个新用户注册的例子。

```java
public class UserRegisterRequest {

    @Pattern(regexp = "1[3-9]{1}[0-9]{9}", message = "the user phone is illegal.")
    private String phone;
    @Email
    private String email;
    @NotBlank(message = "username is required.")
    @Size(min = 1, max = 225, message = "username is too long.")
    private String username;
    @NotBlank(message = "password is required.")
    @Min(value = 8, message = "The new password is too short.")
    private String password;

    //If entity nesting exists, @valid annotation must be added
    @Valid
    @NotNull
    private Address address;
    
    .....
}

public class Address {

    @NotBlank
    private String provice;

    @NotBlank
    private String city;

    @NotBlank
    private String detailAddress;
}

@RestController
public class DemoController {


    @PostMapping("/user/register")
    public Boolean registerUser(@Valid @RequestBody UserRegisterRequest request) {

    }
```

看，就是这么简单。一些注释完成了复杂的验证逻辑。当对目标参数的验证失败时，系统会抛出一个MethodArgumentNotValidException异常。
然后，我们可以使用@ExceptionHandler来处理验证结果。

#### 使用@Exceptionhandler来统一处理验证结果

```java
public class ErrorResult implements Serializable {

    private long code;
    private String msg;

    public long getCode() {
        return code;
    }

    public void setCode(long code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

@Component 
public class GlobalExceptionHandler {  
  
    
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(MethodArgumentNotValidException.class)  
    public ErrorResult parameterExceptionHandler(MethodArgumentNotValidException e) {  
    BindingResult exceptions = e.getBindingResult();  
     if (exceptions.hasErrors()) {  
            List<ObjectError> errors = exceptions.getAllErrors();  
            if (!errors.isEmpty()) {  
                
                FieldError fieldError = (FieldError) errors.get(0); 
                return new ErrorResult(-1,fieldError.getDefaultMessage()); 
                }
          }
          
        return new ErrorResult(-1,"argument valid failure");    
    }  

```

这里创建了一个全局异常处理类。我们使用parameterExceptionHandler方法来捕捉MethodArgumentNotValidException异常，最后，统一处理验证失败的结果。
这个方法将在每个字段的验证失败后得到错误信息。
这里，我使用ErrorResult类来返回错误信息。当我们启动服务时，我们使用postman来发起一个post请求（http://localhost:8080/user/register ），我们不输入任何参数并返回结果。

```json
{  
    "code": -1,  
    "msg": "the user phone is illegal."  
}
```

本文简要介绍了在spring boot中使用@valid来验证API请求。我希望这篇文章能够帮助你。谢谢你的阅读。

<!-- https://medium.com/javarevisited/spring-boot-application-use-valid-to-validate-api-request-f9a328c1ee2c -->