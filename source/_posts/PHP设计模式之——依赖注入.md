---
title: PHP设计模式之——依赖注入
date: 2023-06-08 17:03:44
tags: ["PHP", "设计模式"]
categories: ["技术"]
---

依赖注入(dependency injection)是实现控制反转的一种技术，顾名思义，就是将依赖(被调用者)注入给依赖对象(调用者)。

用户类`User`有一个注册方法，我们需要实现一个功能，在注册成功后，给用户发送一封邮件通知。我们先写一个发送邮件的类：

```PHP
class EmailClass
{
    public function sendEmail($username, $email)
    {
        print_r("send register email to:" . $email . PHP_EOL);
    }
}
```
然后在用户注册方法中实例化一个邮件类：
```PHP
class User
{
    private $username;
    private $password;
    private $mobile;
    private $email;

    public function register()
    {
        // 注册逻辑
        // .........

        // 发送邮件逻辑
        $emailClass = new EmailClass();
        $emailClass->sendEmail($this->username, $this->email);
    }
}
```

这时候又来一个需求，需要在注册成功之后给用户发送短信，于是又要在注册方法里面实例化一个发送短信的类。或者这个时候我们要改一下邮件内容，要把注册密码发给用户，于是我们要给`EmailClass::sendEmail`加一个password参数。经过长期迭代和添油加醋，我们需要频繁的去修改核心注册逻辑，最终的注册方法将变得极为臃肿且难以维护。

<!-- more -->

下面我们使用依赖注入来优化一下这段代码，首先定义一个用于发送注册通知的`interface`：

```PHP
abstract class RegisterNotify
{
    abstract public function send(User $user);
}
```

分别实现发送邮件和短信的类：

```PHP
class RegisterNotifyEmail extends RegisterNotify
{
    public function send(User $user)
    {
        echo "send register email to:" . $user->email . PHP_EOL;
    }
}

class RegisterNotifySms extends RegisterNotify
{
    public function send(User $user)
    {
        echo "send register sms to:" . $user->mobile . PHP_EOL;
    }
}
```

实现用户注册方法：

```PHP
class User
{
    public $username;
    public $password;
    public $mobile;
    public $email;
    public $notifyList = [];

    public function __construct($username, $password, $mobile, $email)
    {
        $this->username = $username;
        $this->password = $password;
        $this->mobile = $mobile;
        $this->email = $email;
    }

    public function setNotify(RegisterNotify $notify)
    {
        $this->notifyList[] = $notify;
    }

    public function register()
    {
        // 注册逻辑
        // .........

        // 发送通知逻辑
        foreach ($this->notifyList as $notify) {
            $notify->send($this);
        }
    }
}
```

最后我们通过如下逻辑来调用：

```PHP
$user = new User('lilei', '123456', '13111111111', 'lilei@gmail.com');
$user->setNotify(new RegisterNotifyEmail($user));
$user->setNotify(new RegisterNotifySms($user));
$user->register();
```

将依赖通过参数从外部业务逻辑处传入，用户类只需要依赖于`RegisterNotify`这一抽象类，并不需要关心具体的通知逻辑是如何实现的，也不需要关心有多少通知。业务逻辑根据需求将不同的消息通知注入到用户类中，而不需要每一次都去修改核心注册逻辑。

常见的依赖注入方式大致可分两种：

 - 方法注入：包括构造方法、成员方法和静态方法传参

 - set属性注入：直接将依赖关系赋值给对象属性

作为一种极为流行的设计模式，依赖注入在各大主力语言中都有着非常广泛的应用，比如PHP的Laravel、Java的Spring，可以充分解耦代码逻辑，提高了代码的可读性和可扩展性。
