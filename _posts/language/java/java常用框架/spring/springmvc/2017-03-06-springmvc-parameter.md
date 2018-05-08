---
title: spring mvc转发和重定向传递参数
tags: [spring]
---

1）重定向传参

使用RedirectAttributes进行参数传递。

```
public String login(@RequestParam Map<String, String> user, 
    RedirectAttributes model) {  
    ....
    model.addFlashAttribute("msg", username);  
    return "redirect:/user/toHome";  
}  
```

2）转发传参

使用ModelAndView即可传递参数

```
public ModelAndView login(@RequestParam Map<String, String> user) {  
    ModelAndView mv=new ModelAndView("forward:/user/toHome");

    ....
    mv.addObject("msg", username);  
    return mv;  
} 
```