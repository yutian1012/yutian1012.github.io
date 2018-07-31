---
title: java异常--实际开发中常用的模式
tags: [interview]
---

在注册用户信息时，当用户输入的信息不满足相应的条件时，可以人为的抛出相应异常信息。

1）定义业务中出现的异常

如邮箱已注册异常，验证用户信息异常和验证密码异常

```
EmailRegisterException 邮箱已注册异常
InvalidAccountInfoException 验证用户信息异常
InvalidPasswordException 验证密码异常
```

注：直接继承Exception，实现自定义异常

2）在service逻辑处理层抛出可能出现的异常

```
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public void register(User user) throws EmailRegisterException,
        InvalidPasswordException, InvalidLoginInfoException {
        //检索到用户邮箱已注册，抛出EmailRegisterException
        if(exists(user.getEmail())){
            throw new EmailRegisterException
        }
        //用户输入账号为空，获取其他不符合数据时抛出异常
        if(null==user.getAccount){
            throw new InvalidAccountInfoException("账号不能输入空！");
        } 
        //用户输入密码为空...
        if(null==user.getPass()) {
            throw new InvalidPasswordException("密码未设置！");
        }
        ....
    }
}
```

注：这里只是描述了一个场景，实际中校验信息使用hibernate validate校验框架即可，没有必要手动的在servcie做校验。这里只是演示业务逻辑抛出自定义异常的实例。

3）在controller层捕获异常并返回相应的response

```
@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserService userService;

    @RequestMapping(method = RequestMethod.GET)
    public ResponseEntity rigister(User user) {
        try {
            user = userService.register(user);
        } catch (EmailNotRegisterException e) {
            //TODO 做邮箱未注册的处理
            return ResponseEntity.status(HttpStatus.FORBIDDEN).body(e.getMessage());
        } catch (InvalidPasswordException e) {
            //TODO 做验证密码失败的处理
            return ResponseEntity.status(HttpStatus.FORBIDDEN).body(e.getMessage());
        } catch (InvalidLoginInfoException e) {
            //TODO 做验证账号失败的处理
            return ResponseEntity.status(HttpStatus.FORBIDDEN).body(e.getMessage());
        }
        return  ResponseEntity.ok().body(user);
    }
}
```