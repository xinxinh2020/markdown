

## Panel

## Dashboard

dashboard是panel的集合。



Query Option

```sh
# All dev pro
# Returns a list of label values for the label in the specified metric
# parm1:metric  parm2:label
label_values(go_memstats_alloc_bytes, namespace) 

# Returns a list of metrics matching the specified metric regex
# 模糊搜索metric
metrics(metric)
```

