---
layout: post
title: "快速构建Android REST客户端#2(待续)"
date: 2013-07-22 17:02
comments: true
categories: 
---
数据解析和存储
===
###JsonPaser
如果你还在使用`Android`提供的`JsonObject`来解析JSON数据，是时候鸟枪换炮了！
Android开发中常用的JsonPaser有这么几个：
* [Gson][1] 由Google开发的Json Parser 使用简单，只需要引入一个jar包，相对于`Android`系统提供的parser，效率有明显的优势
* [Jackson][2] 另一个颇有名气的项目，需要引入很多包相对比较复杂
* [FastJson][3] 阿里主导的开源项目，要求类的每一个属性都具备getter/setter这一点我不能忍
<!--more-->

Gson的使用方法异常简单，例如我们要解析以下数据：
``` json
String json ="{
        "id": 1,
        "name": "Dan Cederholm",
        "username": "simplebits",
        "url": "http://dribbble.com/simplebits",
        "avatar_url": "http://dribbble.com/system/users/1/avatars/original/dancederholm-peek.jpg",
        "location": "Salem, MA",
        "twitter_screen_name": "simplebits",
        "drafted_by_player_id": null,
        "shots_count": 147,
        "draftees_count": 103,
        "followers_count": 2027,
        "following_count": 354,
        "comments_count": 2001,
        "comments_received_count": 1509,
        "likes_count": 7289,
        "likes_received_count": 2624,
        "rebounds_count": 15,
        "rebounds_received_count": 279,
        "created_at": "2009/07/07 21:51:22 -0400"
    }"
```
需要创建Player类，在该类中声明所有需要解析的属性：
``` java
public class Player {
    private long id;
    private String name;
    private String url;
    private String avatar_url;
    private String location;
    private String twitter_screen_name;
    private String drafted_by_player_id;
    private int shots_count;
    private int draftees_count;
    private int followers_count;
    private int following_count;
    private int comments_count;
    private int comments_received_count;
    private int likes_count;
    private int likes_received_count;
    private int rebounds_count;
    private int rebounds_received_count;
    private String created_at;
}
```
使用Gson:
``` java
Gson gson = new Gson();
// Json to object
gson.fromJson(json, Player.class);
// Object to json
String json2 = gson.toJson(Player.class);
```

嘛本着“不会偷懒的程序员不是好程序员”这一遵旨，在`Driibo`的设计中，数据库中仅存储了Json数据，免去了分列的繁琐。缺点是读取数据时需要用Gson重新解析，对效率有一定影响。


###ContentProvier


###CursorAdapter & AsyncTaskLoader


[1]: https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CCsQFjAA&url=http%3a%2f%2fcode%2egoogle%2ecom%2fp%2fgoogle-gson%2f&ei=Vv3sUengEoPJkwXcvYDIDg&usg=AFQjCNGGFFMez8-PfFoEQP93a7eHFY8ssA
[2]: 
[3]: 