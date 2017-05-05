---
title: 开源spagobi--工具类
tags: [spagobi]
---

### 1. GeneralUtilities通用类

1）getSessionExpiredURL方法

通过查询全局配置信息获取session超期的url跳转路径，全局配置表为SBI_CONFIG

### 2. DAOFactory类

### 3. XMLUtil类

### 4. ChannelUtilities类

### 5. UserUtilities类

getUserProfile方法，会调用InternalSecurityServiceSupplierImpl类的createUserProfile方法

```
SbiUser user = DAOFactory.getSbiUserDAO().loadSbiUserByUserId(userId);
ArrayList<SbiExtRoles> rolesSB = DAOFactory.getSbiUserDAO().loadSbiUserRolesById(user.getId());
ArrayList<SbiUserAttributes> attribs = DAOFactory.getSbiUserDAO().loadSbiUserAttributesById(user.getId());
```

注：SpagoBIUserProfile对象封装了用户信息，角色信息以及用户属性等信息。

### 6. PortletUtilities类

/spagobi-core/src/main/java/it/eng/spagobi/security/InternalSecurityServiceSupplierImpl.java
/SpagoBICockpitEngine/src/main/java/it/eng/spagobi/security/InternalSecurityServiceSupplierImpl.java