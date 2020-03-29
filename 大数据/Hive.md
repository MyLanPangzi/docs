## Hive

## idea连接Hive

1. core-site.xml

   ```xml
       <property>
           <!-- atguigu 替换成登录用户名(root，其他用户）-->
           <name>hadoop.proxyuser.atguigu.hosts</name>
           <value>*</value>
       </property>
       <property>
           <name>hadoop.proxyuser.atguigu.groups</name>
           <value>*</value>
       </property>
   
   ```

2. driver![image-20200324100935485](image\idea-hive-driver.png)

3. idea![image-20200324101525433](image\idea-hive.png)