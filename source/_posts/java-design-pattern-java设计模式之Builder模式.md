---
title: '[java][design-pattern]java设计模式之Builder模式'
date: 2016-05-08 16:58:38
tags: [java, 设计模式, Builder, design pattern]
---


## java设计模式之Builder模式

设计模式模式很多，实际常用的很少。
《Effective Java》这本书里提到过的一种模式builder模式，个人认为非常值得推荐。
假设有一个entity(Entity)，有id，name两个字段，如下：
```
package demo;
public class Entity {
    private int id;
    private String name;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```
为了不用每次初始化的时候，要一个一个的setxxx设置字段值，通常我们会创建构造函数，通过构造函数直接传入字段，初始化。
加入构造函数的实体代码如下：
```
public class Entity {
    private int id;
    private String name;
    public Entity() {
        super();
    }
    public Entity(int id, String name) {
        super();
        this.id = id;
        this.name = name;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```
然后需要初始化实体的时候，我们这样做：
```
Entity entity = new Entity(1, "name");
```

这种用法很常用，但是问题来了。
当实体Entity的字段需要增加变化的时候怎么办呢？
比如增加字段descr，这样构造函数和客户端初始化都需要更新：
实体增加：
```
    private String descr;
    public String getDescr() {
        return descr;
    }
    public void setDescr(String descr) {
        this.descr = descr;
    }
    public Entity(int id, String name, String descr) {
        super();
        this.id = id;
        this.name = name;
        this.descr = descr;
    }
```
然后原来调用构造函数初始化的代码也需要改动：
```
Entity entity = new Entity(1, "name", "descr");
```
如果你的class是不确定的参数，后续可能经常变动，那么你的构造函数可能需要很多个，并且不停的变动，而且，构造函数参数多的时候，参数也很不容易记住。

而Effective Java中则推荐一种builder模式来进行实体初始化
如下：
```
public class Entity {
    private int id;
    private String name;
    private String descr;
    public static class Builder {
        private int id;
        private String name;
        private String descr;
        public Builder(int id) {
            this.id = id;
        }
        public Builder name(String name) {
            this.name = name;
            return this;
        }
        public Builder descr(String descr) {
            this.descr = descr;
            return this;
        }
        public Entity build() {
            return new Entity(this);
        }
    }
    private Entity(Builder b) {
        this.id = b.id;
        this.name = b.name;
        this.descr = b.descr;
    }
}
```
初始化实例的时候，如下：
```
Entity entity = new Entity.Builder(10).name("name").descr("descr").build();
```
这样的写法，好处是，假如实体Entity以后变化很大，加入很多字段，不会影响到之前客户端初始化的代码，而且，这个初始化的过程非常清晰简单。
