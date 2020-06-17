

workflow:

```json
[
    {
        "A":{"type":"application","after":[]}
    },
    {
        "B":{"after":["A","D"]}
    },
    {
        "C":{"after":["B"]}
    },
    {
        "D":{"after":["C"]}
    },
    {
        "E":{"after":["D"]}
    },
    {
        "F":{"after":["E"]}
    } 
]
```



runtime:

```json
[
    {
        "A":{
            "count": "1",
            "history":[
            	{"1":[{"index0":"success"}]}
        	]
        }
    },
    {
        "B":{
            "count": "3",
            "history":[
            	{"1":[{"index0":"success"}]},
            	{"2":[{"index0":"success"}]},
            	{"3":[{"index0":"success"}]}
        	]
        }
    },
    {
        "C":{
            "count": "2",
            "history":[
            	{"1":[{"index0":"success"},{"index1":"running"},{"index2":"success"}]},
            	{"2":[{"index0":"success"},{"index1":"running"},{"index2":"success"}]}
        	]
        }
    },
    {
        "D":{
            "count": "2",
            "history":[
            	{"1":[{"index0":"success"}]},
            	{"2":[{"index0":"success"}]}
        	]
        }
    }
]
```

