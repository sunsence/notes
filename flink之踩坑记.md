# flink之踩坑记 --flink高可用配置

##                                           --flink高可用配置

配置信息：
 high-availability: zookeeper

 high-availability.storageDir: hdfs:///flink/ha/

 high-availability.zookeeper.quorum: host1:2181,host2:2181,host3:2181

 high-availability.zookeeper.path.root: /flink



1. 启动方式与配置

当high-availability: zookeeper，选择使用flink run -m的方式提交job时，-m option会被忽略掉（参考flink client源码），导致提交job失败。会出现如下错误信息：

```
2019-02-27 11:48:38,651 INFO  org.apache.flink.runtime.rest.RestClient                      - Shutting down rest endpoint.
2019-02-27 11:48:38,659 INFO  org.apache.flink.runtime.rest.RestClient                      - Rest endpoint shutdown complete.
2019-02-27 11:48:38,662 INFO  org.apache.flink.runtime.leaderretrieval.ZooKeeperLeaderRetrievalService  - Stopping ZooKeeperLeaderRetrievalService /leader/rest_server_lock.
2019-02-27 11:48:38,665 INFO  org.apache.flink.runtime.leaderretrieval.ZooKeeperLeaderRetrievalService  - Stopping ZooKeeperLeaderRetrievalService /leader/dispatcher_lock.
2019-02-27 11:48:38,670 INFO  org.apache.flink.shaded.curator.org.apache.curator.framework.imps.CuratorFrameworkImpl  - backgroundOperationsLoop exiting
2019-02-27 11:48:38,689 INFO  org.apache.flink.shaded.zookeeper.org.apache.zookeeper.ZooKeeper  - Session: 0x2679c52880c00ee closed
2019-02-27 11:48:38,689 INFO  org.apache.flink.shaded.zookeeper.org.apache.zookeeper.ClientCnxn  - EventThread shut down for session: 0x2679c52880c00ee
2019-02-27 11:48:38,690 ERROR org.apache.flink.client.cli.CliFrontend                       - Error while running the command.
org.apache.flink.client.program.ProgramInvocationException: Could not retrieve the execution result.
          at org.apache.flink.client.program.rest.RestClusterClient.submitJob(RestClusterClient.java:257)
          at org.apache.flink.client.program.ClusterClient.run(ClusterClient.java:464)
          at org.apache.flink.streaming.api.environment.StreamContextEnvironment.execute(StreamContextEnvironment.java:66)
          at org.apache.flink.streaming.api.scala.StreamExecutionEnvironment.execute(StreamExecutionEnvironment.scala:654)
          at scala.Function0$class.apply$mcV$sp(Function0.scala:34)
          at scala.runtime.AbstractFunction0.apply$mcV$sp(AbstractFunction0.scala:12)
          at scala.App$$anonfun$main$1.apply(App.scala:76)
          at scala.App$$anonfun$main$1.apply(App.scala:76)
          at scala.collection.immutable.List.foreach(List.scala:381)
          at scala.collection.generic.TraversableForwarder$class.foreach(TraversableForwarder.scala:35)
          at scala.App$class.main(App.scala:76)
          at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
          at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
          at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
          at java.lang.reflect.Method.invoke(Method.java:498)
          at org.apache.flink.client.program.PackagedProgram.callMainMethod(PackagedProgram.java:528)
          at org.apache.flink.client.program.PackagedProgram.invokeInteractiveModeForExecution(PackagedProgram.java:420)
          at org.apache.flink.client.program.ClusterClient.run(ClusterClient.java:404)
          at org.apache.flink.client.cli.CliFrontend.executeProgram(CliFrontend.java:785)
          at org.apache.flink.client.cli.CliFrontend.runProgram(CliFrontend.java:279)
          at org.apache.flink.client.cli.CliFrontend.run(CliFrontend.java:214)
          at org.apache.flink.client.cli.CliFrontend.parseParameters(CliFrontend.java:1025)
          at org.apache.flink.client.cli.CliFrontend.lambda$main$9(CliFrontend.java:1101)
          at java.security.AccessController.doPrivileged(Native Method)
          at javax.security.auth.Subject.doAs(Subject.java:422)
          at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1754)
          at org.apache.flink.runtime.security.HadoopSecurityContext.runSecured(HadoopSecurityContext.java:41)
          at org.apache.flink.client.cli.CliFrontend.main(CliFrontend.java:1101)
Caused by: org.apache.flink.runtime.client.JobSubmissionException: Failed to submit JobGraph.
          at org.apache.flink.client.program.rest.RestClusterClient.lambda$submitJob$8(RestClusterClient.java:370)
          at java.util.concurrent.CompletableFuture.uniExceptionally(CompletableFuture.java:870)
          at java.util.concurrent.CompletableFuture$UniExceptionally.tryFire(CompletableFuture.java:852)
          at java.util.concurrent.CompletableFuture.postComplete(CompletableFuture.java:474)
          at java.util.concurrent.CompletableFuture.completeExceptionally(CompletableFuture.java:1977)
          at org.apache.flink.runtime.concurrent.FutureUtils.lambda$retryOperationWithDelay$5(FutureUtils.java:214)
          at java.util.concurrent.CompletableFuture.uniWhenComplete(CompletableFuture.java:760)
          at java.util.concurrent.CompletableFuture$UniWhenComplete.tryFire(CompletableFuture.java:736)
          at java.util.concurrent.CompletableFuture.postComplete(CompletableFuture.java:474)
          at java.util.concurrent.CompletableFuture.completeExceptionally(CompletableFuture.java:1977)
          at org.apache.flink.runtime.concurrent.FutureUtils$Timeout.run(FutureUtils.java:834)
          at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
          at java.util.concurrent.FutureTask.run(FutureTask.java:266)
          at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
          at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
          at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
          at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
          at java.lang.Thread.run(Thread.java:748)
Caused by: java.util.concurrent.CompletionException: org.apache.flink.runtime.concurrent.FutureUtils$RetryException: Could not complete the operation. Exception is not retryable.
          at java.util.concurrent.CompletableFuture.encodeRelay(CompletableFuture.java:326)
          at java.util.concurrent.CompletableFuture.completeRelay(CompletableFuture.java:338)
          at java.util.concurrent.CompletableFuture.uniRelay(CompletableFuture.java:911)
          at java.util.concurrent.CompletableFuture$UniRelay.tryFire(CompletableFuture.java:899)
          ... 15 more
Caused by: org.apache.flink.runtime.concurrent.FutureUtils$RetryException: Could not complete the operation. Exception is not retryable.
          ... 13 more
Caused by: java.util.concurrent.CompletionException: java.util.concurrent.TimeoutException
          at java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:292)
          at java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:308)
          at java.util.concurrent.CompletableFuture.uniApply(CompletableFuture.java:593)
          at java.util.concurrent.CompletableFuture$UniApply.tryFire(CompletableFuture.java:577)
          ... 10 more
Caused by: java.util.concurrent.TimeoutException
          ... 8 more
 
------------------------------------------------------------
 The program finished with the following exception:
 
org.apache.flink.client.program.ProgramInvocationException: Could not retrieve the execution result.
          at org.apache.flink.client.program.rest.RestClusterClient.submitJob(RestClusterClient.java:257)
          at org.apache.flink.client.program.ClusterClient.run(ClusterClient.java:464)
          at org.apache.flink.streaming.api.environment.StreamContextEnvironment.execute(StreamContextEnvironment.java:66)
          at org.apache.flink.streaming.api.scala.StreamExecutionEnvironment.execute(StreamExecutionEnvironment.scala:654)
          at scala.Function0$class.apply$mcV$sp(Function0.scala:34)
          at scala.runtime.AbstractFunction0.apply$mcV$sp(AbstractFunction0.scala:12)
          at scala.App$$anonfun$main$1.apply(App.scala:76)
          at scala.App$$anonfun$main$1.apply(App.scala:76)
          at scala.collection.immutable.List.foreach(List.scala:381)
          at scala.collection.generic.TraversableForwarder$class.foreach(TraversableForwarder
          at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
          at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
          at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
          at java.lang.reflect.Method.invoke(Method.java:498)
          at org.apache.flink.client.program.PackagedProgram.callMainMethod(PackagedProgram.java:528)
          at org.apache.flink.client.program.PackagedProgram.invokeInteractiveModeForExecution(PackagedProgram.java:420)
          at org.apache.flink.client.program.ClusterClient.run(ClusterClient.java:404)
          at org.apache.flink.client.cli.CliFrontend.executeProgram(CliFrontend.java:785)
          at org.apache.flink.client.cli.CliFrontend.runProgram(CliFrontend.java:279)
          at org.apache.flink.client.cli.CliFrontend.run(CliFrontend.java:214)
          at org.apache.flink.client.cli.CliFrontend.parseParameters(CliFrontend.java:1025)
          at org.apache.flink.client.cli.CliFrontend.lambda$main$9(CliFrontend.java:1101)
          at java.security.AccessController.doPrivileged(Native Method)
          at javax.security.auth.Subject.doAs(Subject.java:422)
          at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1754)
          at org.apache.flink.runtime.security.HadoopSecurityContext.runSecured(HadoopSecurityContext.java:41)
          at org.apache.flink.client.cli.CliFrontend.main(CliFrontend.java:1101)
Caused by: org.apache.flink.runtime.client.JobSubmissionException: Failed to submit JobGraph.
          at org.apache.flink.client.program.rest.RestClusterClient.lambda$submitJob$8(RestClusterClient.java:370)
          at java.util.concurrent.CompletableFuture.uniExceptionally(CompletableFuture.java:870)
          at java.util.concurrent.CompletableFuture$UniExceptionally.tryFire(CompletableFuture.java:852)
          at java.util.concurrent.CompletableFuture.postComplete(CompletableFuture.java:474)
          at java.util.concurrent.CompletableFuture.completeExceptionally(CompletableFuture.java:1977)
          at org.apache.flink.runtime.concurrent.FutureUtils.lambda$retryOperationWithDelay$5(FutureUtils.java:214)
          at java.util.concurrent.CompletableFuture.uniWhenComplete(CompletableFuture.java:760)
          at java.util.concurrent.CompletableFuture$UniWhenComplete.tryFire(CompletableFuture.java:736)
          at java.util.concurrent.CompletableFuture.postComplete(CompletableFuture.java:474)
          at java.util.concurrent.CompletableFuture.completeExceptionally(CompletableFuture.java:1977)
          at org.apache.flink.runtime.concurrent.FutureUtils$Timeout.run(FutureUtils.java:834)
          at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
          at java.util.concurrent.FutureTask.run(FutureTask.java:266)
          at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
          at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
          at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
          at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
          at java.lang.Thread.run(Thread.java:748)
Caused by: java.util.concurrent.CompletionException: org.apache.flink.runtime.concurrent.FutureUtils$RetryException: Could not complete the operation. Exception is not retryable.
          at java.util.concurrent.CompletableFuture.encodeRelay(CompletableFuture.java:326)
          at java.util.concurrent.CompletableFuture.completeRelay(CompletableFuture.java:338)
          at java.util.concurrent.CompletableFuture.uniRelay(CompletableFuture.java:911)
          at java.util.concurrent.CompletableFuture$UniRelay.tryFire(CompletableFuture.java:899)
          ... 15 more
Caused by: org.apache.flink.runtime.concurrent.FutureUtils$RetryException: Could not complete the operation. Exception is not retryable.
          ... 13 more
Caused by: java.util.concurrent.CompletionException: java.util.concurrent.TimeoutException
          at java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:292)
          at java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:308)
          at java.util.concurrent.CompletableFuture.uniApply(CompletableFuture.java:593)
          at java.util.concurrent.CompletableFuture$UniApply.tryFire(CompletableFuture.java:577)
          ... 10 more
Caused by: java.util.concurrent.TimeoutException
          ... 8 more
```

因为flink高可用需要用zookeeper进行jm的leader选举，所以是必须配置的。

解决办法：修改提交方式为flink run -yid  appid my.jar的方式就可以正常提交了。通过指定appid，可以将Job提交到对应的yarn cluster上。



2. 使用flink run -yid  appid my.jar的问题

```
java.lang.NoClassDefFoundError: com/sun/jersey/core/util/FeaturesAndProperties
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
        at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
        at java.net.URLClassLoader.defineClass(URLClassLoader.java:467)
        at java.net.URLClassLoader.access$100(URLClassLoader.java:73)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:368)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:362)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:361)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        at org.apache.hadoop.yarn.client.api.TimelineClient.createTimelineClient(TimelineClient.java:55)
        at org.apache.hadoop.yarn.client.api.impl.YarnClientImpl.createTimelineClient(YarnClientImpl.java:181)
        at org.apache.hadoop.yarn.client.api.impl.YarnClientImpl.serviceInit(YarnClientImpl.java:168)
        at org.apache.hadoop.service.AbstractService.init(AbstractService.java:163)
        at org.apache.flink.yarn.cli.FlinkYarnSessionCli.getClusterDescriptor(FlinkYarnSessionCli.java:984)
        at org.apache.flink.yarn.cli.FlinkYarnSessionCli.createDescriptor(FlinkYarnSessionCli.java:273)
        at org.apache.flink.yarn.cli.FlinkYarnSessionCli.createClusterDescriptor(FlinkYarnSessionCli.java:451)
        at org.apache.flink.yarn.cli.FlinkYarnSessionCli.createClusterDescriptor(FlinkYarnSessionCli.java:96)
        at org.apache.flink.client.cli.CliFrontend.runProgram(CliFrontend.java:224)
        at org.apache.flink.client.cli.CliFrontend.run(CliFrontend.java:213)
        at org.apache.flink.client.cli.CliFrontend.parseParameters(CliFrontend.java:1050)
        at org.apache.flink.client.cli.CliFrontend.lambda$main$11(CliFrontend.java:1126)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1754)
        at org.apache.flink.runtime.security.HadoopSecurityContext.runSecured(HadoopSecurityContext.java:41)
        at org.apache.flink.client.cli.CliFrontend.main(CliFrontend.java:1126)
Caused by: java.lang.ClassNotFoundException: com.sun.jersey.core.util.FeaturesAndProperties
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
```

原因：使用这种方式提交job时会用到flink-shaded-hadoop2-uber-1.7.2.jar .在flink的源码中可以看到，此包的pom中把相关的jersey全部都exclude掉了，导致缺包。

解决办法： 将对应版本的jersey.core的jar放到$FLINK_HOME/lib下



3. 其他可能遇到的问题

hdfs目录权限的问题。

原因：flink需要将高可用相关信息写入到配置的hdfs目录下，如果没有写权限，也会出现提交失败的问题。

        Caused by: org.apache.hadoop.security.AccessControlException: Permission denied: user=root, access=WRITE, inode="/flink/ha/application_1545373844579_0333/blob":hdfs:hdfs:drwxr-xr-x
            at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.check(FSPermissionChecker.java:353)
            at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.check(FSPermissionChecker.java:325)
            at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:246)
            at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:190)
            at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1950)
            at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1934)
            at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkAncestorAccess(FSDirectory.java:1917)
            at org.apache.hadoop.hdfs.server.namenode.FSDirMkdirOp.mkdirs(FSDirMkdirOp.java:71)
            at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:4181)
            at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:1109)
            at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.mkdirs(ClientNamenodeProtocolServerSideTranslatorPB.java:645)
            at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
            at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:640)
            at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:982)
            at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2351)
            at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2347)
            at java.security.AccessController.doPrivileged(Native Method)
            at javax.security.auth.Subject.doAs(Subject.java:422)
            at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1869)
            at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2347)
        
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
        at org.apache.hadoop.ipc.RemoteException.instantiateException(RemoteException.java:106)
        at org.apache.hadoop.ipc.RemoteException.unwrapRemoteException(RemoteException.java:73)
        at org.apache.hadoop.hdfs.DFSClient.primitiveMkdir(DFSClient.java:3016)
        at org.apache.hadoop.hdfs.DFSClient.mkdirs(DFSClient.java:2984)
        at org.apache.hadoop.hdfs.DistributedFileSystem$21.doCall(DistributedFileSystem.java:1047)
        at org.apache.hadoop.hdfs.DistributedFileSystem$21.doCall(DistributedFileSystem.java:1043)
        at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirsInternal(DistributedFileSystem.java:1061)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirs(DistributedFileSystem.java:1036)
        at org.apache.hadoop.fs.FileSystem.mkdirs(FileSystem.java:1880)
        at org.apache.flink.runtime.fs.hdfs.HadoopFileSystem.mkdirs(HadoopFileSystem.java:170)
        at org.apache.flink.runtime.blob.FileSystemBlobStore.<init>(FileSystemBlobStore.java:61)
        at org.apache.flink.runtime.blob.BlobUtils.createFileSystemBlobStore(BlobUtils.java:126)
        at org.apache.flink.runtime.blob.BlobUtils.createBlobStoreFromConfig(BlobUtils.java:92)
        at org.apache.flink.runtime.highavailability.HighAvailabilityServicesUtils.createHighAvailabilityServices(HighAvailabilityServicesUtils.java:121)
        at org.apache.flink.client.program.ClusterClient.<init>(ClusterClient.java:159)
        at org.apache.flink.client.program.rest.RestClusterClient.<init>(RestClusterClient.java:185)
        at org.apache.flink.client.program.rest.RestClusterClient.<init>(RestClusterClient.java:158)
        at org.apache.flink.yarn.YarnClusterDescriptor.createYarnClusterClient(YarnClusterDescriptor.java:96)
        at org.apache.flink.yarn.AbstractYarnClusterDescriptor.retrieve(AbstractYarnClusterDescriptor.java:401)
