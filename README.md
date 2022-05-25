# Streams GroupBy Reduce java 8

Purpose : to group objects having `status` equal to my `customizedStatus` to a single customized one with `count` = `sumOfSameObjectsCount` <br/>
Input : 
```
[
                        {
                            "id": 1,
                            "name": "nameXX",
                            "status": "statusXX",
                            "count": 1
                        },
                        {
                            "id": 2,
                            "name": "nameYY",
                            "status": "statusYY",
                            "count": 2
                        },
                        {
                            "id": 3,
                            "name": "nameZZ",
                            "status": "customizedStatus",
                            "count": 8
                        },
                        {
                            "id": 4,
                            "name": "nameZY",
                            "status": "customizedStatus",
                            "count": 2
                        }
                    ]
```
Expected output : 
```
[
                        {
                            "id": 1,
                            "name": "nameXX",
                            "status": "statusXX",
                            "count": 1
                        },
                        {
                            "id": 2,
                            "name": "nameYY",
                            "status": "statusYY",
                            "count": 2
                        },
                        {
                            "id": customizedId,
                            "name": "customizedName",
                            "status": "customizedStatus",
                            "count": 10
                        }
                    ]
```
We have class MyObject : 
```
class MyObject {
   Integer id;
   String name;
   String status;
   Long count;
   //constructor with attributes
   //getters 
   //setters
} 
```
Suggested stream list preparing peace 
```
listOfObjects.stream()
		.collect(Collectors.groupingBy(Object::getStatus))
		.entrySet().stream()
		.map(e -> e.getValue().stream()
				.reduce((partialResult,nextElem) -> 
					{
						System.out.println("ahaaaa! inside your reduce block ");
						if(partialResult.getStatus().equals(customizedStatus)) {
              System.out.println("in case of tobeaproved status");
							return new Object(customizedId, customizedName, customizedStatus, partialResult.getCount()+nextElem.getCount());
						} else {
              System.out.println("in case of ! tobeaproved status");
							return new Object(partialResult.getId(), partialResult.getName(), partialResult.getStatus(), partialResult.getCount());
						}
					}
				)
			)	
		.collect(Collectors.toList());
 ```

1. With this solution, things work like a charm in case there are ***multiple objects*** with `status` equal to `customizedStatus`, means in case of `groupingBy` groups entries by `customizedStatus` -***reduce is executed***- <br/>
As a result, entries with `status` equal to `customizedStatus` are being customized. 
2. In case of list entries are diffrent by `status`, we have 2 problems : <br/>
	- Reduce would be skipped in case of `groupingBy` doesn't group by `status`. <br/>
	- Reduce is not skipped, but doesn't return all the objects. <br/>

Are we missing something !! **True**, the missing piece is the **Identity** [reduce documentation](https://www.baeldung.com/java-stream-reduce)
> Identity – an element that is the initial value of the reduction operation and the default result if the stream is empty <br/>

For that solution would to initialize first our reduce and keep iterating over the map ! <br/> 

```
public static MyObject getIdentity(Map.Entry<String, List<MyObject>> entry) {
        
    return entry.getKey().equals(customizedStatus) ?
            new MyObject(customizedId, customizedName, customizedStatus, 0L) :
            entry.getValue().iterator().next();
    }
    
    public static MyObject accumulate(MyObject result, MyObject next) {
        
    return result.getStatus().equals(customizedStatus) ?
            new MyObject(customizedId, customizedName, customizedStatus, result.getCount() + next.getCount()) :
            new MyObject(result.getId(), result.getName(), result.getStatus(), result.getCount());
    }
```
You can play around with this [Online Demo](https://www.jdoodle.com/ia/r9U) <br/>
```
import java.util.*;
import java.util.stream.Collectors;


public class MyClass {
    
     static Integer customizedId = 99;
         static String customizedName = "customizedName";
         static String customizedStatus = "customizedStatus";
        
        
    public static void main(String[] args) {
    List<MyObject> listOfObjects =
        List.of(new MyObject(1, "nameXX", "statusXX", 1L),
                new MyObject(2, "nameYY", "statusYY", 1L),
                new MyObject(12, "nameZZz", "customizedStatus", 3L),
                new MyObject(12, "nameZZAZE", "customizedStatus", 3L)
                );
    
    List<MyObject> result =
        listOfObjects.stream()
            .collect(Collectors.groupingBy(MyObject::getStatus))
            .entrySet().stream()
            .map(e -> e.getValue().stream()
                .reduce(getIdentity(e), (partialResult, nextElem) -> accumulate(partialResult, nextElem)) )
            .collect(Collectors.toList());
        
    result.forEach(System.out::println);
    
    

}

public static MyObject getIdentity(Map.Entry<String, List<MyObject>> entry) {
        
    return entry.getKey().equals(customizedStatus) ?
            new MyObject(customizedId, customizedName, customizedStatus, 0L) :
            entry.getValue().iterator().next();
    }
    
    public static MyObject accumulate(MyObject result, MyObject next) {
        
    return result.getStatus().equals(customizedStatus) ?
            new MyObject(customizedId, customizedName, customizedStatus, result.getCount() + next.getCount()) :
            new MyObject(result.getId(), result.getName(), result.getStatus(), result.getCount());
    }


}


class MyObject {
        Integer id;
        String name;
        String status;
        Long count;
    
        public MyObject(Integer id, String name, String status, Long count) {
            this.id = id;
            this.name = name;
            this.status = status;
            this.count = count;
        }
    
        public Integer getId() {
            return id;
        }
    
        public String getName() {
            return name;
        }
    
        public String getStatus() {
            return status;
        }
    
        public Long getCount() {
            return count;
        }
    
        @Override
        public String toString() {
            return "MyObject{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", status='" + status + '\'' +
                ", count=" + count +
                '}';
        }
    }
```
That's it ! your `listOfObjects` now is grouped and customized as it is , as you wish ! And take advantage of your reduce block to do whatever you want.
Done
