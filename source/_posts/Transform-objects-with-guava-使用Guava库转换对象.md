---
title: Transform objects with guava(使用Guava库转换对象)
date: 2016-07-23 13:03:29
tags: [java, guava, transform objects, convert, 对象转换]
---

# Transform objects with guava

All the function is a specific type of class that has an apply method that can be overridden. In the example, the apply method accepts a string as a parameter and returns an integer or the length of the string. Scrolling a bit further in the documentation, the most common use of functions is transforming collections.
```
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
  public Integer apply(String string) {
    return string.length();
  }
};
```


## Convert string to Enum



I have seeded some data so we can get right to the examples and you don't have to watch me type. The first example to show is transforming a string to Enum. Enums are quite popular and you may want to transform as it might be stored as a string in a database or some value.

Taking a look at the Day enum we created, it is a simple class that represents the days of the week:
```
public enum Day {

    SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY, SATURDAY;   
}
```
We may have a list of strings with various strings representing days, Wednesday, Sunday, Monday... What we want to do is convert the list of strings to enums. There is a function in the Enum utility class in guava valueOfFunction that allows you to pass in the enum you want to convert to. In our case, Day.Class. Since we just upgraded to Guava 16.0, the valueOfFunction has been deprecated in preference of stringConverter. The stringConverter will return a function that converts a string to an Enum. Now we have a function that can be passed to guava utilities, in this case Iterables utility class by calling the transform method which will convert each string of list to a day enum.
```
@Test
public void transform_string_to_enum () {
    
    List<String> days = Lists.newArrayList(
            "WEDNESDAY", 
            "SUNDAY", 
            "MONDAY", 
            "WEDNESDAY");
    
    Function<String, Day> stringToDayEnum = Enums.stringConverter(Day.class);
    
    Iterable<Day> daysAsEnum = Iterables.transform(days, stringToDayEnum);
    
    for (Day day : daysAsEnum) {
        System.out.println(day);
    }
}
```
Output
```
WEDNESDAY
SUNDAY
MONDAY
WEDNESDAY
```



## Convert from one object to another

The next example will look at is how to convert one object to another. Quite often we need to go to a legacy system to get data and populate a new set of domains for our new system. Or we may get data from a web service or whatever it may be. For this example, I created two objects. ETradeInvestment and TdAmeritradeInvestment which contain similar attributes of different types.
```
public class ETradeInvestment {
    
    private String key;
    private String name;
    private BigDecimal price;

    ...
}

public class TdAmeritradeInvestment {
    
    private int investmentKey;
    private String investmentName;
    private double investmentPrice;

    ...
}
```
There is a number of other utilities such as BeanUtils.copyProperties in apache commons or written your own if statements and have made it work that way. If we don't have a list of objects, we can call a method on function that will return a just a single object. We want to create the function that will map the TdAmeritradeInvestment to ETradeInvestment. If you ever get lost, you can use code assist or just remember the <F, T> just means from object, to object. For each iteration of the list, a new ETradeInvestment object will be created while mapping the TdAmeritradeInvestment to it. For the key, we will use the Ints.stringConverter which allows us to convert from a string to an int.

If we want to get some reuse out of this function, we could extract it outside this method or we can just use it inline. We will use the Lists utility to transform the list, above we use Iterables and there is also FluentIterables and Collections2. There is a lot of different ways which guava provides to use a function. If we run this code, we have EtradeInvestment object with the key, name and price. As a disclosure, I don't own any of these investments and were pulled from Google home page under the trending section. That did it, we converted from the TdAmeritradeInvestment to ETradeInvestment.
```
@Test
public void convert_tdinvestment_etradeinvestment () {
    
    List<TdAmeritradeInvestment> tdInvestments = Lists.newArrayList();
    tdInvestments.add(new TdAmeritradeInvestment(555, "Facebook Inc", 57.51));
    tdInvestments.add(new TdAmeritradeInvestment(123, "Micron Technology, Inc.", 21.29));
    tdInvestments.add(new TdAmeritradeInvestment(456, "Ford Motor Company", 15.31));
    tdInvestments.add(new TdAmeritradeInvestment(236, "Sirius XM Holdings Inc", 3.60));
    
    // convert a list of objects
    Function<TdAmeritradeInvestment, ETradeInvestment> tdToEtradeFunction = new Function<TdAmeritradeInvestment, ETradeInvestment>() {

        public ETradeInvestment apply(TdAmeritradeInvestment input) {
            ETradeInvestment investment = new ETradeInvestment();
            investment.setKey(Ints.stringConverter().reverse()
                    .convert(input.getInvestmentKey()));
            investment.setName(input.getInvestmentName());
            investment.setPrice(new BigDecimal(input.getInvestmentPrice()));
            return investment;
        }
    };

    List<ETradeInvestment> etradeInvestments = Lists.transform(tdInvestments, tdToEtradeFunction);
    
    System.out.println(etradeInvestments);
}
```
Output
```
[
ETradeInvestment{key=555, name=Facebook Inc, price=57.50},
ETradeInvestment{key=123, name=Micron Technology, Inc., price=21.28}, 
ETradeInvestment{key=456, name=Ford Motor Company, price=15.31}, 
ETradeInvestment{key=236, name=Sirius XM Holdings Inc, price=3.60}
]
```



## Convert an object

If you have one single investment or object you want to convert, you can call the apply method directly on the function that will do the conversion.

ETradeInvestment faceBookInvestment = tdToEtradeFunction
                .apply(new TdAmeritradeInvestment(555, "Facebook Inc", 57.51));
Output
```
ETradeInvestment{key=555, name=Facebook Inc, price=57.50}
```



## Convert list to map

One other really neat way to use function convert a list to a map. You may want to look up an object based on some object property. In this example, we want to create a map of TdAmeritradeInvestment based on the investment key. Taking a look we can use the Maps utility calling the uniqueIndex method which accepts a list and a function. The function, or the keyfunction, is used to produce the key for each value in the iterable. In this instance, the function will map from a TdAmeritradeInvestment and return an integer which will represent the key in the map.
```
@Test
public void transform_list_to_map () {
    
    List<TdAmeritradeInvestment> tdInvestments = Lists.newArrayList();
    tdInvestments.add(new TdAmeritradeInvestment(555, "Facebook Inc", 57.51));
    tdInvestments.add(new TdAmeritradeInvestment(123, "Micron Technology, Inc.", 21.29));
    tdInvestments.add(new TdAmeritradeInvestment(456, "Ford Motor Company", 15.31));
    tdInvestments.add(new TdAmeritradeInvestment(236, "Sirius XM Holdings Inc", 3.60));
    
    ImmutableMap<Integer, TdAmeritradeInvestment> investmentMap = Maps
            .uniqueIndex(tdInvestments,
                    new Function<TdAmeritradeInvestment, Integer>() {

                        public Integer apply(TdAmeritradeInvestment input) {
                            return new Integer(input.getInvestmentKey());
                        }
                    });
    
    System.out.println(investmentMap);
    
}
````
Output
```
{
555=TdAmeritradeInvestment{key=555, name=Facebook Inc, price=57.51}, 
123=TdAmeritradeInvestment{key=123, name=Micron Technology, Inc., price=21.29}, 
456=TdAmeritradeInvestment{key=456, name=Ford Motor Company, price=15.31}, 
236=TdAmeritradeInvestment{key=236, name=Sirius XM Holdings Inc, price=3.6}
}
```

原地址：http://www.leveluplunch.com/java/tutorials/002-transform-objects-with-guava/