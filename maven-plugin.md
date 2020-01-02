# Maven Plugin Build

## 创建项目

## 复制pom.xml

**注意这个 <packaging>maven-plugin</packaging>**

```xml

    <groupId>com.hiscat</groupId>
    <artifactId>hello-maven-plugin</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>maven-plugin</packaging>

    <dependencies>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-plugin-api</artifactId>
            <version>3.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugin-tools</groupId>
            <artifactId>maven-plugin-annotations</artifactId>
            <version>3.4</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

## install

## 时间戳插件

```xml
			<finalName>ProjectName-${current.time}</finalName>  
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>build-helper-maven-plugin</artifactId>
                <version>1.9.1</version>
                <executions>
                    <execution>
                        <id>timestamp-property</id>
                        <goals>
                            <goal>timestamp-property</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <name>current.time</name>
                    <pattern>yyyyMMdd-HHmmss</pattern>
                    <timeZone>GMT+8</timeZone>
                </configuration>
            </plugin>
```

