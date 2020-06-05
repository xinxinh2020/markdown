



## 常用代码示例

调用shell命令，获取返回码和输出

```python
import subprocess
(status, output)=subprocess.getstatusoutput('ls -alh')
print(status)
print(output)
```

打开文件

```python
startlight_apis_file=open("startlight_apis.txt", mode='r')
try:
    for line in startlight_apis_file:
         print(line) # line带"\n"
finally:
     startlight_apis_file.close()
```

