---
title: github开源项目云收藏（右侧工具栏）
tags: [project]
---

sidebar.html页面使用到了${config}，首先要明确config实体对象的作用。

1）Config实体对象

可配置用户的默认收藏夹，收藏夹是否公开等信息。默认用户注册时会创建一个用户的配置对象，并将未读列表作为默认收藏夹。因此，用户注册完成后会生成favorites对象（未读列表收藏夹）和config对象（用户全局配置对象）。

2）创建相关的实体对象、service和dao层类

a.创建Config实体类

```
package com.favorites.favoriteswebdemo.domain;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Setter
@Getter
@NoArgsConstructor
public class Config implements Serializable{
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false)
    private Long userId;
    @Column(nullable = false)
    private String defaultFavorties;
    @Column(nullable = false)
    private String defaultCollectType;
    @Column(nullable = false)
    private String defaultModel;
    @Column(nullable = false)
    private Long createTime;
    @Column(nullable = false)
    private Long lastModifyTime;
}
```

b.创建favorites实体类

```
package com.favorites.favoriteswebdemo.domain;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Getter
@Setter
@NoArgsConstructor
public class Favorites implements Serializable{
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false)
    private Long userId;
    @Column(nullable = false)
    private String name;
    @Column(nullable = false)
    private Long count;
    @Column(nullable = false)
    private Long createTime;
    @Column(nullable = false)
    private Long lastModifyTime;
    @Column(nullable = false)
    private Long publicCount;
}
```

c.创建ConfigService接口和实现类

```
```

