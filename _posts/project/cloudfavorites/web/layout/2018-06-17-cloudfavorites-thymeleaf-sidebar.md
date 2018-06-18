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
package com.favorites.favoriteswebdemo.service;

import com.favorites.favoriteswebdemo.domain.Config;

public interface ConfigService {
    public Config saveConfig(Long userId,String favoritesId);

    public void updateConfig(long id ,String type,String defaultFavorites);
    
    public Config findByUserId(Long userId);
}
```

实现类

```
package com.favorites.favoriteswebdemo.service;

import javax.annotation.Resource;
import javax.transaction.Transactional;

import org.springframework.stereotype.Service;

import com.favorites.favoriteswebdemo.dao.ConfigRepository;
import com.favorites.favoriteswebdemo.domain.Config;

@Service
@Transactional
public class ConfigServiceImpl implements ConfigService{
    
    @Resource
    private ConfigRepository configRepository;

    @Override
    public Config saveConfig(Long userId, String favoritesId) {
        Config config = new Config();
        config.setUserId(userId);
        config.setDefaultModel("simple");
        config.setDefaultFavorties(favoritesId);
        config.setDefaultCollectType("public");
        config.setCreateTime(System.currentTimeMillis());
        config.setLastModifyTime(System.currentTimeMillis());
        configRepository.save(config);
        return config;
    }

    @Override
    public void updateConfig(long id, String type, String defaultFavorites) {
        Config config = configRepository.findById(id);
        String value="";
        if("defaultCollectType".equals(type)){
            if("public".equals(config.getDefaultCollectType())){
                value = "private";
            }else{
                value = "public";
            }
            configRepository.updateCollectTypeById(id, value,System.currentTimeMillis());
        }else if("defaultModel".equals(type)){
            if("simple".equals(config.getDefaultModel())){
                value = "major";
            }else{
                value="simple";
            }
            configRepository.updateModelTypeById(id, value, System.currentTimeMillis());
        }else if("defaultFavorites".equals(type)){
            configRepository.updateFavoritesById(id, defaultFavorites,System.currentTimeMillis());
        }
    }

    @Override
    public Config findByUserId(Long userId) {
        return configRepository.findByUserId(userId);
    }
}
```

ConfigRepository类

```
package com.favorites.favoriteswebdemo.dao;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;

import com.favorites.favoriteswebdemo.domain.Config;

public interface ConfigRepository extends JpaRepository<Config, Long>{
    Config findByUserId(Long userId);

    Config findById(long id);

    Config findByUserIdAndDefaultFavorties(Long userId,String defaultFavorites);
    
    @Modifying
    @Query("update Config set defaultCollectType=?2,lastModifyTime =?3 where id = ?1")
    int updateCollectTypeById(Long id,String value,Long lastModifyTime);
    
    @Modifying
    @Query("update Config set defaultModel=?2,lastModifyTime =?3 where id = ?1")
    int updateModelTypeById(Long id,String value,Long lastModifyTime);
    
    @Modifying
    @Query("update Config set defaultFavorties=?2,lastModifyTime =?3 where id = ?1")
    int updateFavoritesById(Long id,String value,Long lastModifyTime);
}
```

d.创建Favorites收藏夹相关的操作类

```
package com.favorites.favoriteswebdemo.service;

import com.favorites.favoriteswebdemo.domain.Favorites;

public interface FavoritesService {
    
    public Favorites findById(Long id);
    
    public Favorites saveFavorites(Long userId,String name);
}
```

FavoritesServiceImpl实现类

```
package com.favorites.favoriteswebdemo.service;

import javax.annotation.Resource;
import javax.transaction.Transactional;

import org.springframework.stereotype.Service;

import com.favorites.favoriteswebdemo.dao.FavoritesRepository;
import com.favorites.favoriteswebdemo.domain.Favorites;

@Service
@Transactional
public class FavoritesServiceImpl implements FavoritesService{
    
    @Resource
    private FavoritesRepository favoritesRepository;

    @Override
    public Favorites saveFavorites(Long userId, String name) {
        Favorites favorites = new Favorites();
        favorites.setName(name);
        favorites.setUserId(userId);
        favorites.setCount(0l);
        favorites.setPublicCount(10l);
        favorites.setCreateTime(System.currentTimeMillis());
        favorites.setLastModifyTime(System.currentTimeMillis());
        favoritesRepository.save(favorites);
        return  favorites;
    }

    @Override
    public Favorites findById(Long id) {
        return favoritesRepository.findById(id.longValue());
    }

}
```

FavoritesRepository类

```
package com.favorites.favoriteswebdemo.dao;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;

import com.favorites.favoriteswebdemo.domain.Favorites;

public interface FavoritesRepository extends JpaRepository<Favorites, Long>{
    
    Favorites findById(long  id);

    List<Favorites> findByUserId(Long userId);

    List<Favorites> findByUserIdOrderByIdDesc(Long userId);

    Favorites findByUserIdAndName(Long userId,String name);

}
```

3）修改注册逻辑

用户注册成功后即生成相关的Config实体和Favorites默认收藏夹

修改UserServiceImpl类的save方法

```
@Override
public User save(User user) {
    userRepository.save(user);
    // 添加默认收藏夹
    Favorites favorites = favoritesService.saveFavorites(user.getId(), "未读列表");
    // 添加默认属性设置
    configService.saveConfig(user.getId(),String.valueOf(favorites.getId()));
    
    return user;
}
```

注：并把操作放置在同一个事务中。这里坚持的一个原则，对于修改操作，尽量在service层间调用不同的服务，而不是将其他模块的service注入到controller。

4）修改登录逻辑

用户登录后，需要加载用户的收藏夹对象和全局配置对象。

修改IndexController类的home方法

```
@RequestMapping(value="/",method=RequestMethod.GET)
public String home(Model model) {
    Config config = configService.findByUserId(getUserId());
    Favorites favorites = favoritesService.findById(Long.parseLong(config.getDefaultFavorties()));
    model.addAttribute("config",config);
    model.addAttribute("favorites",favorites);
    model.addAttribute("user",getUser());
    return "home";
}
```

注：这里将favoritesService和configService注入到了IndexController类中，因为这里只是涉及到查询操作，但也不提倡使用这种方式。建议还是有一个统一的service作为门面来实现调用。

注2：鉴于IndexController类并没有与之相关的Service和实体类，因此这里可暂且简单注入其他模块的service。