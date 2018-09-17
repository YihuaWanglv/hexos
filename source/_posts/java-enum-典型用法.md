---
title: '[java][enum]典型用法'
date: 2016-05-08 23:57:31
tags: [java, enum]
categories: java
---


## java enum典型用法

### 代码：
```
package com.iyihua.model.enums;

import java.util.HashMap;
import java.util.Map;

public enum GroupType {

     CATEGORY(0, "category"), PROJECT(1, "project"), LOCATION(2, "location" );

     private final int id;
     private final String key;

     GroupType(int id, String key) {
            this. id = id;
            this. key = key;
     }
     
     public int getId() {
            return id;
     }
     public String getKey() {
            return key;
     }

     private static final Map<String, GroupType> keyToEnum = new HashMap<String, GroupType>();
     static {
            for (GroupType gt : GroupType. values())
                 keyToEnum.put(gt .getKey(), gt );
     }

     public static GroupType fromString(String symbol) {
            return keyToEnum.get(symbol );
     }

     public static void main(String[] args) {
           System. out.println( fromString("category"));
     }
}
```


### 说明：
代码中把枚举值对应的string-枚举放到了一个map keyToEnum中，并且是静态化的。
如此就可以通过fromString方法，传入key string值，即可返回对应的enum值