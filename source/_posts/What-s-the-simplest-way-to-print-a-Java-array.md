---
title: What's the simplest way to print a Java array?
date: 2016-08-08 18:24:43
tags: [java, array, print, j2se]
---


Examples:
- Simple Array:
```
String[] array = new String[] {"John", "Mary", "Bob"};
System.out.println(Arrays.toString(array));
```
Output:
```
[John, Mary, Bob]
```

- Nested Array:
```
String[][] deepArray = new String[][] {{"John", "Mary"}, {"Alice", "Bob"}};
System.out.println(Arrays.toString(deepArray));
//output: [[Ljava.lang.String;@106d69c, [Ljava.lang.String;@52e922]
System.out.println(Arrays.deepToString(deepArray));
```
Output:
```
[[John, Mary], [Alice, Bob]]
```

- double Array:
```
double[] doubleArray = { 7.0, 9.0, 5.0, 1.0, 3.0 };
System.out.println(Arrays.toString(doubleArray));
```
Output:
```
[7.0, 9.0, 5.0, 1.0, 3.0 ]
```
- int Array:
```
int[] intArray = { 7, 9, 5, 1, 3 };
System.out.println(Arrays.toString(intArray));
```
Output:
```
[7, 9, 5, 1, 3 ]
```