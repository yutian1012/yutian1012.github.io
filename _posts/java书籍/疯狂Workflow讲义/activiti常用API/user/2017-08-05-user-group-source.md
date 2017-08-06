---
title: 用户组管理源码查看
tags: [activiti]
---

### 创建Group对象，底层返回的是GroupEntity实例

1）newGroup方法

查看IdentityServiceImpl类的newGroup方法

```
public Group newGroup(String groupId) {
    return commandExecutor.execute(new CreateGroupCmd(groupId));
}
```

2）最终是由CreateGroupCmd命令对象来执行的

查看CreateGroupCmd类的execute方法

```
public Group execute(CommandContext commandContext) {
    return commandContext.getGroupManager().createNewGroup(groupId);
}
```

通过获取CommandContext上下文环境，得到GroupManager，然后创建对象。

3）查看GroupManager创建组对象

```
public Group createNewGroup(String groupId) {
    return new GroupEntity(groupId);
}
```

注：可以看到该类new出来GroupEntity对象，并返回。并没有直接将对象保存到数据库中。

### 保存Group对象

1）saveGroup方法

查看IdentityServiceImpl类的saveGroup方法

```
public void saveGroup(Group group) {
    commandExecutor.execute(new SaveGroupCmd((GroupEntity) group));
}
```

2）最终是由SaveGroupCmd命令对象来执行的

查看SaveGroupCmd类的execute方法

```
public Void execute(CommandContext commandContext) {
    if(group == null) {
      throw new ActivitiException("group is null");
    }
    //通过判断group对象的Revision字段来判断是更新还是删除
    if (group.getRevision()==0) {
      commandContext
        .getGroupManager()
        .insertGroup(group);
    } else {
      commandContext
        .getGroupManager()
        .updateGroup(group);
    }
    
    return null;
}
```

3）查看GroupManger的insertGroup方法

```
public void insertGroup(Group group) {
    getDbSqlSession().insert((PersistentObject) group);
}
```

注：这里直接将Group对象强转成了PersistentObject对象，原因是GroupEntity实现了PersistentObject接口。

4）获取Session

GroupManager类继承了AbstractManger抽象类，GroupManager类实现了更具体对象的封装操作。

getDbSqlSession方法的实现位于AbstractManager类中，获取DbSqlSessionFactory，然后调用openSession获取Session对象（DbSqlSession），最后执行insert方法。

5）执行数据库操作

DbSqlSession类中定义了数据库的操作，如insert，delete等。
