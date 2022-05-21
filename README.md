# streams

I'm trying to customize my collection stream using groupedBy and reduce, I have a list of objects that I need to be grouped in case of similar status.

We have class Object
```
class Object {
   String status;
   String status_label;
   String status_id;
   Long count;
   //constructor with attributes
   //getters 
   //setters
} 
```
stream list preparing peace 
```
listOfObjects.stream()
		.collect(Collectors.groupingBy(Object::getStatus))
		.entrySet().stream()
		.map(e -> e.getValue().stream()
				.reduce((partialResult,nextElem) -> 
					{
						LOGGER.info("ahaaaa! inside your reduce block ");
						if(partialResult.getStatus().equals(customizedStatus)) {
              LOGGER.info("in case of tobeaproved status");
							return new Object(customizedId, customizedName, customizedStatus, partialResult.getCount()+nextElem.getCount());
						} else {
              LOGGER.info("in case of ! tobeaproved status");
							return new Object(partialResult.getId(), partialResult.getName(), partialResult.getStatus(), partialResult.getCount());
						}
					}
				)
			)
		.collect(Collectors.toList());
 ```
1. Things work like a charm in case if there are objects with status in common.<br/>
Input : 
```
[
                        {
                            "status": "processed",
                            "status_label": "Processed",
                            "status_id": 63,
                            "count": 33
                        },
                        {
                            "status": "tobecompleted",
                            "status_label": "To be completed",
                            "status_id": 50,
                            "count": 2095
                        },
                        {
                            "status": "entryinprogress",
                            "status_label": "To be completed",
                            "status_id": 58,
                            "count": 8
                        },
                        {
                            "status": "tobesent",
                            "status_label": "To be sent by client",
                            "status_id": 52,
                            "count": 33
                        },
                        {
                            "status": "transferredtoagency",
                            "status_label": "To be processed by supplier",
                            "status_id": 54,
                            "count": 60
                        },
                        {
                            "status": "tobeapproved",
                            "status_label": "To be approved",
                            "status_id": 51,
                            "count": 70
                        }
                        {
                            "status": "tobeapproved",
                            "status_label": "To be approved 3",
                            "status_id": 65,
                            "count": 15
                        }
                    ]
```
Output : 
![image](https://user-images.githubusercontent.com/59615955/169663496-82bc2661-f0b6-4364-a21a-534077a37926.png)
As you can see, reduce is executed after logging out the other entries with no status in common.<br/>
Entries with status in common are being customized with the new status label and count. 
```
[
                        {
                            "status": "processed",
                            "status_label": "Processed",
                            "status_id": 63,
                            "count": 33
                        },
                        {
                            "status": "tobecompleted",
                            "status_label": "To be completed",
                            "status_id": 50,
                            "count": 2095
                        },
                        {
                            "status": "entryinprogress",
                            "status_label": "To be completed",
                            "status_id": 58,
                            "count": 8
                        },
                        {
                            "status": "tobesent",
                            "status_label": "To be sent by client",
                            "status_id": 52,
                            "count": 33
                        },
                        {
                            "status": "transferredtoagency",
                            "status_label": "To be processed by supplier",
                            "status_id": 54,
                            "count": 60
                        },
                        {
                            "status": "tobeapproved",
                            "status_label": "To be approved",
                            "status_id": 51,
                            "count": 85
                        }
                    ]
```		    
2. In case of list entries are diffrent by status.<br/>
Input : 
```
[
                        {
                            "status": "processed",
                            "status_label": "Processed",
                            "status_id": 63,
                            "count": 33
                        },
                        {
                            "status": "tobecompleted",
                            "status_label": "To be completed",
                            "status_id": 50,
                            "count": 2095
                        },
                        {
                            "status": "entryinprogress",
                            "status_label": "To be completed",
                            "status_id": 58,
                            "count": 8
                        },
                        {
                            "status": "tobesent",
                            "status_label": "To be sent by client",
                            "status_id": 52,
                            "count": 33
                        },
                        {
                            "status": "transferredtoagency",
                            "status_label": "To be processed by supplier",
                            "status_id": 54,
                            "count": 60
                        },
                        {
                            "status": "tobeapproved",
                            "status_label": "To be approved 3",
                            "status_id": 65,
                            "count": 15
                        }
                    ]
```
![image](https://user-images.githubusercontent.com/59615955/169663910-d3fbb6bd-1d5a-44e4-8c0d-c2402b56fdd0.png)
As you can see reduce block is being skipped ! our object that needs to be customized unfortunately it is not ! we still have it with the old status label; <br/>
Output :
```
[
                        {
                            "status": "processed",
                            "status_label": "Processed",
                            "status_id": 63,
                            "count": 33
                        },
                        {
                            "status": "tobecompleted",
                            "status_label": "To be completed",
                            "status_id": 50,
                            "count": 2095
                        },
                        {
                            "status": "entryinprogress",
                            "status_label": "To be completed",
                            "status_id": 58,
                            "count": 8
                        },
                        {
                            "status": "tobesent",
                            "status_label": "To be sent by client",
                            "status_id": 52,
                            "count": 33
                        },
                        {
                            "status": "transferredtoagency",
                            "status_label": "To be processed by supplier",
                            "status_id": 54,
                            "count": 60
                        },
                        {
                            "status": "tobeapproved",
                            "status_label": "To be approved 3",
                            "status_id": 65,
                            "count": 15
                        }
                    ]
```
