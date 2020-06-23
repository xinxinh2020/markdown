



## 配置maven源



```xml
<repositories>
        <repository>
            <id>alimaven</id>
            <name>Maven Aliyun Mirror</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

