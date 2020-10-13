---
title: Java 异常处理之 Web 案例
date: 2018-07-19T08:00:23.000Z
tags: ['Java', 'Web']
categories: [Coding]
---
## 前言

基于我的上一篇博客：[Java 异常处理](https://s1mple.online/2018/07/16/Java%20%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)中的观点，我编写了一个简单的 Web 案例，包括自定义异常类，在合适的层面处理，以及遇到的由于代码 Bug 抛出运行时异常的情形。示例代码已经上传至 [GitHub](https://github.com/s1mplecc/exception-demo)，包含基于 Spring Boot 的 Java 后端代码以及基于 Vue.js 的前端代码两个部分。
 
我们的演示案例有两个功能，获取用户列表以及注册新用户，界面如下所示：

![102BDB82-6CAD-4C3F-A5EC-5F1052F3A337](/0.png)

## 后端实现

先看一下我们领域模型 User，为简化代码未使用数据库，而是模拟数据库的自增主键自动生成 userId。

```java
@Data
public class User {

    private static int userIdGenerateKey = 1;

    private Integer userId;
    private String username;
    private String phone;
    private Integer age;

    public User() {
    }

    User(String username, String phone, Integer age) {
        this.userId = User.userIdGenerateKey++;
        this.username = username;
        this.phone = phone;
        this.age = age;
    }

    User(User userWithoutId) {
        this(userWithoutId.username, userWithoutId.phone, userWithoutId.age);
    }
}
```

在资源库`UserRepository`一层使用`List`直接在内存中存储用户，并初始化了两条用户信息。对外提供`users()`返回所有用户以及`addUser()`添加一个新用户两个接口。

```java
@Repository
public class UserRepository {

    private List<User> users = new ArrayList<>();

    public UserRepository() {
        users.add(new User("Jack", "13500000000", 18));
        users.add(new User("Tom", "13500000001", 19));
    }

    public List<User> users() {
        return users;
    }

    public void addUser(User user) {
        users.add(user);
    }
}
```
    
考虑到在业务功能为获取用户列表和注册新用户，而注册功能还可能包括发送验证邮件等之类的功能，所以有必要将其提取至 Service 层提供一个`register()`接口。目前只是单纯的调用资源库的`addUser()`添加一个新的用户。

```java
@Service
public class UserService {

    @Resource
    private UserRepository userRepository;

    public void register(User user) {
        userRepository.addUser(new User(user));
    }
}
```

Controller 层代码如下：

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @Resource
    private UserRepository userRepository;
    @Resource
    private UserService userService;

    @GetMapping
    public List<User> users() {
        return userRepository.users();
    }

    @PostMapping
    public void register(@RequestBody User user) {
        userService.register(user);
    }
}
```
    
### 添加业务规则进行校验

上述案例还是未抛出任何异常的情况，实际业务必然会对用户输入做一定约束。假设注册新用户有如下业务规则：1. 用户名是必填项，不能为空；2. 年龄和电话号码为可选项，如果填入年龄，必须是大于零的合法数字。

那么根据 DDD 的思想以及信息专家模式，检查用户名和年龄是否合法应该属于领域对象 User。如下所示，这里先埋个坑，也是我未思考全面所留下的隐患，一会儿会提及。

```java
// User

public boolean checkAgeIsLegal() {
    return this.age >= 1;
}

public boolean checkUsernameIsNotEmpty() {
    return username != null && !"".equals(username.trim());
}
```

### 抛出异常

如果客户输入的值不符合要求，我们应当抛出一个异常提醒用户及时修正。自定义`RequestArgsIllegalException`异常类，意为请求参数非法，重写`getMessage()`方法。异常类的命名要清晰准确反应问题。

```java
public class RequestArgsIllegalException extends Exception {
    public RequestArgsIllegalException(String message) {
        super(message);
    }

    public RequestArgsIllegalException(String message, Throwable cause) {
        super(message, cause);
    }

    @Override
    public String getMessage() {
        return "Illegal request arguments: " + super.getMessage();
    }
}
```

在服务层的`register()`接口，我们使用**卫语句**检查参数，如果非法则抛出包含了异常信息的异常。

```java
// UserService

public void register(User user) throws RequestArgsIllegalException {
    if (!user.checkAgeIsLegal()) {
        throw new RequestArgsIllegalException("age cannot be less than zero");
    }
    if (!user.checkUsernameIsNotEmpty()) {
        throw new RequestArgsIllegalException("username can not be empty");
    }

    userRepository.addUser(new User(user));
}
```

在 Controller 层捕获异常并将异常信息返回给前端：

```java
// UserController

@PostMapping
public void register(@RequestBody User user, HttpServletResponse response) throws IOException {
    try {
        userService.register(user);
    } catch (RequestArgsIllegalException e) {
        logger.warn("{}. {}", e.getMessage(), user);
        response.sendError(HttpServletResponse.SC_BAD_REQUEST, e.getMessage());
    }
}
```

## 接口调试

使用 Postman 测试后端`register()`接口，请求体如下，其中年龄违反了业务规则。

```json
{
    "username": "Halen",
    "age": -1,
    "phone": "13500000002"
}
```

通过 Postman 返回的错误信息，状态吗`status`为 400，错误信息`message`为 "Illegal request arguments: age cannot be less than zero" 都已经被包装在返回的消息体中。同理，用户名为空也是一样就不贴出来了。

```json
{
    "timestamp": 1532055599002,
    "status": 400,
    "error": "Bad Request",
    "message": "Illegal request arguments: age cannot be less than zero",
    "path": "/users"
}
```

这里又要提到我之前说到的那个坑了，也是我在使用 Postman 测试接口时发现的 Bug。根据业务规则允许用户不填入年龄和电话号码。

```json
// 请求消息体
{
    "username": "Halen"
}

// 响应消息体
{
    "timestamp": 1532056232417,
    "status": 500,
    "error": "Internal Server Error",
    "message": "No message available",
    "path": "/users"
}
```

非常可怕的 500 服务器内部错误出现了！根据响应消息完全无法定位错误出处。一般来说，这种情况是后端代码出了 Bug，所幸我们还能查看控制台输出或者日志。果不其然，一个空指针异常被抛出：

```
java.lang.NullPointerException: null
    at info.s1mple.exceptiondemo.domian.model.user.User.checkAgeIsLegal(User.java:30)
```

原来是`checkAgeIsLegal()`方法出了问题，赶紧去及时修复 Bug，根据业务规则，添加`this.age == null`认为年龄为空合法。

```java
public boolean checkAgeIsLegal() {
    return this.age == null || this.age >= 1;
}
```

## 前端实现

前端实现就很简单了，捕获到错误直接 alert 出来。前端使用 axios 发送异步请求，请求后端的注册用户接口，`this.user`是绑定了用户输入表单值的 User 对象。如果参数符合要求提示成功，如果参数非法则会走入`catch`分支将错误原因提醒给用户。注意到 Error 对象是做了封装的，我们要获取可以在`err.response.data`中获取。

```js
register() {
    axios.post('http://localhost:8080/users', this.user)
      .then(() => {
        alert('Success!')
        this.loadUsers()
      })
      .catch((err) => {
        alert(err.response.data.message)
      })
  }
```
    
前端捕获了异常将异常出错信息弹框显示：

![06A6A59C-1FDF-4BA6-9DDB-6D29BDD489A9](/1.png)

## 总结

由此我们整个的前后端异常处理的过程就贯通了，在这个案例中我使用的是受控异常（Checked Exception）并且自定义了异常类。可以看到，相较于运行时异常（RuntimeException），它的好处在于能够很好的将异常分门别类，并且捕获异常返回相应的异常码给前端，也方便前端的报错呈现。最后我们总结一下：**对于由用户操作引起的异常，在我们的控制范围之内，应当使用合适的受控异常对其进行捕获并反馈给前端**。
