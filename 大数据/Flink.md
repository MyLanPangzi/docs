# Flink

## Maven

```shell
mvn archetype:generate                               \
      -DarchetypeGroupId=org.apache.flink              \
      -DarchetypeArtifactId=flink-quickstart-scala     \
      -DarchetypeVersion=1.10.0   \
      -DgroupId=frauddetection \
      -DartifactId=frauddetection \
      -Dversion=0.1 \
      -Dpackage=spendreport \
      -DinteractiveMode=false
      
 mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-datastream-scala \
    -DarchetypeVersion=1.10.0 \
    -DgroupId=frauddetection \
    -DartifactId=frauddetection \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false
```

```xml
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-scala_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
```

## Basic API Concepts

### DataSet and DataStream

### Anatomy of a Flink Program

1. Obtain an `execution environment`,
2. Load/create the initial data,
3. Specify transformations on this data,
4. Specify where to put the results of your computations,
5. Trigger the program execution