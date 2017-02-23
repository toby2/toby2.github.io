## Array和链表的性能对比
***

#### **Gradle代码**
***
```gradle
group 'com.toby'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'war'
apply plugin: 'idea'

sourceCompatibility = 1.8

repositories {
    mavenLocal();
    mavenCentral()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.5.1.RELEASE")
    compile("org.projectlombok:lombok:1.16.10")
    testCompile group: 'junit', name: 'junit', version: '4.11'
    compile group: 'org.openjdk.jmh', name: 'jmh-core', version: '1.17.4'
    compile group: 'org.openjdk.jmh', name: 'jmh-generator-annprocess', version: '1.17.4'
    compile group: 'com.goldmansachs', name: 'gs-collections', version: '7.0.3'
}
```
#### **测试代码**
***
```java

package com.toby.awesome1.collection;

import com.toby.awesome1.jmh.SecondBenchmark;
import lombok.extern.slf4j.Slf4j;
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
@State(Scope.Benchmark)
@Slf4j
public class CompareArrayLink {
    List array;
    List link;
    String[] dataArray = {"1", "2", "3", "4", "5", "6", "7", "8","9","10","1", "2", "3", "4", "5", "6", "7", "8","9","10"};

    /*@Benchmark
    public void insertNodeToArray() {
        array = new ArrayList();
        for (String data : dataArray) {
            array.add(data);
        }
        array = null;
    }

    @Benchmark
    public void insertNodeToLink() {
        link = new LinkedList();
        for (String data : dataArray) {
            link.add(data);
        }
        link = null;
    }*/

    @Benchmark
    public void getNodeFromArray(){
        //log.info("\n getNodeFromArray...............");
        for(int i=0;i<dataArray.length;i++){
            array.get(i);
        }
    }

    @Benchmark
    public void getNodeFromLink(){
        //log.info("\n getNodeFromLink...............");
        for(int i=0;i<dataArray.length;i++){
            link.get(i);
        }
    }

    @Setup
    public void initData(){
        array = new ArrayList();
        link = new LinkedList();
        for (String data : dataArray) {
            array.add(data);
        }
        for (String data : dataArray) {
            link.add(data);
        }
        log.info("\n setup...............");
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(CompareArrayLink.class.getSimpleName())
                .forks(1)
                .warmupIterations(5)
                .measurementIterations(5)
                .build();

        new Runner(opt).run();
    }

}
```
#### **测试结果**
***
| Benchmark                          | Mode| Cnt|  Score | Error |Units|
| ---------------------------------- |-----| -- | -----: |--     |--   |
| CompareArrayLink.getNodeFromArray|thrpt| 5|290720314.908|± 5739248.915|ops/s|
|CompareArrayLink.getNodeFromLink |  thrpt|    5|   18752661.768 |±  106270.497  |ops/s|
|CompareArrayLink.insertNodeToArray|  thrpt|    5 | 7240786.772| ± 218585.373 | ops/s|
|CompareArrayLink.insertNodeToLink|   thrpt|    5|  8148276.550| ± 145529.979 | ops/s|
#### **结论**
***
ArrayList

以数组实现。节约空间，但数组有容量限制。超出限制时会增加50%容量，用System.arraycopy（）复制到新的数组。因此最好能给出数组大小的预估值。默认第一次插入元素时创建大小为10的数组。

按数组下标访问元素－get（i）、set（i,e） 的性能很高，这是数组的基本优势。

如果按下标插入元素、删除元素－add（i,e）、 remove（i）、remove（e），则要用System.arraycopy（）来复制移动部分受影响的元素，性能就变差了。

越是前面的元素，修改时要移动的元素越多。直接在数组末尾加入元素－常用的add（e），删除最后一个元素则无影响。

LinkedList

以双向链表实现。链表无容量限制，但双向链表本身使用了更多空间，每插入一个元素都要构造一个额外的Node对象，也需要额外的链表指针操作。

按下标访问元素－get（i）、set（i,e） 要悲剧的部分遍历链表将指针移动到位 （如果i>数组大小的一半，会从末尾移起）。

插入、删除元素时修改前后节点的指针即可，不再需要复制移动。但还是要部分遍历链表的指针才能移动到下标所指的位置。

只有在链表两头的操作－add（）、addFirst（）、removeLast（）或用iterator（）上的remove（）倒能省掉指针的移动。

Apache Commons 有个TreeNodeList，里面是棵二叉树，可以快速移动指针到位。
#### **相关链接**
***
http://calvin1978.blogcn.com/articles/collection.html
http://openjdk.java.net/projects/code-tools/jmh/