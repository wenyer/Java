## @Autowired的使用，推荐对构造方法进行注释

以下是：@Autowired和构造方法执行的**顺序**解析

先看一段代码，下面的代码能运行成功吗？

```java
@Autowired
private User user;
private String school;

public UserAccountServiceImpl(){
    this.school = user.getSchool();
}
```

答案是不能。

因为Java类会先执行构造方法，然后再给注解了@Autowired 的user注入值，所以在执行构造方法的时候，就会报错。

解决办法是，使用构造器注入，如下：

```java
private User user;
private String school;

@Autowired
public UserAccountServiceImpl(User user){
    this.user = user;
    this.school = user.getSchool();
}
```

可以看出，使用构造器注入的方法，<font color=red>**可以明确成员变量的加载顺序**。</font>

Java变量的初始化顺序为：静态变量或静态语句块–>实例变量或初始化语句块–>构造方法–>@Autowired