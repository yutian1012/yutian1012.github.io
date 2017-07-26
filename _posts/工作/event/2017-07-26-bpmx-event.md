---
title: bpmx系统中的事件机制
tags: [work]
---

内部使用了spring的事件机制，如用户激活事件。系统监听该事件，执行用户激活的相关操作。如用户激活后，需要将用户信息与系统的专利信息进行关联，因此，可以使用事件监听，在监听中执行相关操作（也可以看作是一种异步的处理方式）。

1）定义事件

事件对象需要继承ApplicationEvent（org.springframework.context.ApplicationEvent）类。

```
public class SysUserActiveImportPatentEvent extends ApplicationEvent{
    
    private Long userId;
    public SysUserActiveImportPatentEvent(Long source) {
        super(source);
        this.userId=source;
    }
    public Long getUserId() {
        return userId;
    }
    public void setUserId(Long userId) {
        this.userId = userId;
    }
}
```

2）触发事件（事件源）

代码逻辑中需要判断用户的更新信息中，是用户被激活，如果被激活，则触发相应的激活事件。

事件的触发对象可称为事件源，如button按钮的点击事件，button按钮对象就是事件源。点下的操作会触发事件的生成和发布。

```
//发布用户激活消息
if(sysuser.isActived()){//判断是否用户被激活了
    EventUtil.userActiveImportPatent(sysUser.getUserId());
}
```

EventUtil是一个工具类，该类实现事件发布，供监听事件执行监听。

```
/**
 * 发布用户激活事件
 * @param userId
 */
public static void userActiveImportPatent(Long userId){
    SysUserActiveImportPatentEvent event=new SysUserActiveImportPatentEvent(userId);
    AppUtil.publishEvent(event);
}
```

构造出相应的事件对象，然后发布该事件。

```
public class AppUtil implements ApplicationContextAware {

    private static  ApplicationContext applicationContext;
    /**
     * Spring发布事件。
     * @param applicationEvent
     */
    public static void publishEvent(ApplicationEvent applicationEvent){
        applicationContext.publishEvent(applicationEvent);
    }
    ...
}
```

注：内部使用applicationContext发布事件。发布事件即通知监听，会调用监听集合的所有监听，然后执行监听接口方法。

3）事件监听

事件监听器实现ApplicationListener接口，并实现onApplicationEvent方法，在方法中编写事件相关的处理方法。

```
/**
 * 用户激活后自动导入用户的专利信息
 */
@Service
public class SysUserActiveImportPatentEventHandler implements ApplicationListener<SysUserActiveImportPatentEvent>{

    @Override
    public void onApplicationEvent(SysUserActiveImportPatentEvent event) {
        SysUserService sysUserService=AppUtil.getBean(SysUserService.class);
        PatentImportAssist patentImportAssist=AppUtil.getBean(PatentImportAssist.class);
        SysUser user=sysUserService.getById(event.getUserId());
        if(null!=user){
            try {
                patentImportAssist.importPatentByPatentee(user.getFullname());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```