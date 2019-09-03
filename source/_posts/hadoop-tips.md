---
title: hadoop tips
date: 2019-04-19 09:53:13
tags:
    - hadoop
categories:
    - Distributed System
    - Bigdata Technology
---

# hadoop tips

- yarn查看application log的方式：

```shell
1.在${HADOOP_HOME}/etc/hadoop/yarn-site.xml中配置：
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>
<property >
<name>yarn.log-aggregation.retain-seconds</name>
<value>604800</value>
</property>
<property>
<name>yarn.nodemanager.remote-app-log-dir</name>
<value>/app-logs</value>
</property>
<property>
<name>yarn.nodemanager.remote-app-log-dir-suffix</name>
<value>logs</value>
</property>
<!-- 是否将对容器强制实施物理内存限制 -->
<property>
<name>yarn.nodemanager.pmem-check-enabled</name>
<value>false</value>
</property>
<!-- 是否将对容器强制实施虚拟内存限制 -->
<property>
<name>yarn.nodemanager.vmem-check-enabled</name>
<value>false</value>
</property>

2.在hdfs上创建/app-logs
hadoop fs -mkdir /app-logs

3.在/app-logs下查看运行过的application的applicationId，然后使用如下命令查看日志：
yarn logs -applicationId application_12**345
```

- hadoop关于jobhistory server的配置，以便在作业web界面查看日志信息

```hadoop
1.mapred-site.xml的配置
<property>
<name>mapreduce.jobhistory.address</name>
<value>localhost:10020</value>
</property>
<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value>localhost:19888</value>
</property>
2.yarn-site.xml的配置
</property>
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>
<property>
<name>yarn.log.server.url</name>
<value>http://localhost:19888/jobhistory/logs/</value>
```

- hadoop启动方式:

```
1.启动hdfs
./sbin/start-dfs.sh
或者分开启动
./sbin/hadoop-daemon.sh start namenode
./sbin/hadoop-daemon.sh start datanode
2.启动yarn
./sbin/start-yarn.sh
或者分开启动
./sbin/yarn-daemon.sh start resourcemanager
./sbin/yarn-daemon.sh start nodemanager
3.启动jobhistory server
./sbin/mr-jobhistory-daemon.sh start historyserver
```

- hadoop jar提交作业时提示Exception in thread "main" java.lang.SecurityException: Invalid signature file digest for Manifest main attributes，跟jar包签名有关，可以在.pom下配置plugin:

```maven
<groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                     </execution>
                </executions>
```

如果还无法解决问题，就在每次生成jar包后，命令行下输入以下命令删除签名文件:

```shell
zip -d wordcount.jar 'META-INF/.SF' 'META-INF/.RSA' 'META-INF/*SF'
```

- hdfs的目录默认是放在/user/linux用户名下的
- 关闭namenode的safemode:hdfs dfsadmin -safemode leave
