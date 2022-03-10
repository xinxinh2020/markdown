

## 资源对象转换

### runtime.Object 转 map[string]interface{}

```
originUnstructuredMap, err := runtime.DefaultUnstructuredConverter.ToUnstructured(obj)
```



### runtime.Object 转 unstructured.Unstructured

```
obj.(*unstructured.Unstructured)
```



## 微服务



![image-20211215154543981](images/k8s/image-20211215154543981.png)