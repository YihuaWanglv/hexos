---
title: How to synchronize ArrayList in java with example
date: 2016-07-28 13:21:15
tags: [java, synchronize, ArrayList]
categories: java
---


## There are two ways to synchronize explicitly:

- Using Collections.synchronizedList() method
- Using thread-safe variant of ArrayList: CopyOnWriteArrayList

## Example 1: Collections.synchronizedList() method for Synchronizing ArrayList

In this example we are using Collections.synchronizedList() method. The important point to note here is that iterator should be in synchronized block in this type of synchronization as shown in the below example.

```
package beginnersbook.com;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Collections;

public class Details {

   public static void main(String a[]){
       List<String> syncal = 
         Collections.synchronizedList(new ArrayList<String>());

       //Adding elements to synchronized ArrayList
       syncal.add("Pen");
       syncal.add("NoteBook");
       syncal.add("Ink");

       System.out.println("Iterating synchronized ArrayList:");
       synchronized(syncal) {
       Iterator<String> iterator = syncal.iterator(); 
       while (iterator.hasNext())
          System.out.println(iterator.next());
       }
   }
}
```
- Output:
```
Iterating synchronized ArrayList:
Pen
NoteBook
Ink
```


## Method 2: Using CopyOnWriteArrayList

CopyOnWriteArrayList is a thread-safe variant of ArrayList.

```
package beginnersbook.com;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.Iterator;

public class Details {

 public static void main(String a[]){
    CopyOnWriteArrayList<String> al = new CopyOnWriteArrayList<String>();

    //Adding elements to synchronized ArrayList
    al.add("Pen");
    al.add("NoteBook");
    al.add("Ink");

    System.out.println("Displaying synchronized ArrayList Elements:");
    //Synchronized block is not required in this method
    Iterator<String> iterator = al.iterator(); 
    while (iterator.hasNext())
       System.out.println(iterator.next());
  }
}
```
- Output:
```
Displaying synchronized ArrayList Elements:
Pen
NoteBook
Ink
```