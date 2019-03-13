# Flink 踩坑记

##                                                          --coding见招拆招🤪

1. ### Row类型中的字段类型必须显示声明，包括以下两种解决方案

2. 1. Implicit   Types.ROW() 会在声明的整个作用域内生效，执行flink sql之后可能导致schema不匹配，无法从table转为ds。可以通过代码块的方式解决

3. `{`

4. `Implicit   Types.ROW()`

5. `}`

6. `{`

7. `Implicit   Types.ROW()`

8. `}`

9. Yes, that's a workaround. I found the cause of the problem. It  is a Scala API specific problem.

10. See: <https://issues.apache.org/jira/browse/FLINK-9556>

11. 2. 可以通过ds.map()Types.ROW()的方式定义row中的字段类型

12. 

13. ###  反序列化

14. 需要自定义序列化，如何把kafka接收到的数据进行反序列化，需要显示指定返回数据类型



1. ### Lazy  execution

2. All Flink programs  are executed lazily: When the program’s main method is executed, the data  loading and  transformations do not happen directly. Rather, each operation is  created and added to the program’s plan. The operations are actually executed  when the execution is explicitly triggered by an execute() call on the execution environment. Whether the program is  executed locally or on a cluster depends on the type of execution environment

3. The lazy  evaluation lets you construct sophisticated programs that Flink executes as  one holistically planned unit.

4. 先生成执行计划，Evn.execute()才会真正触发执行

5. 

6. ### I have a NotSerializableException

7. Flink  uses Java serialization to distribute copies of the application logic (the  functions and operations you implement, as well as the program configuration,  etc.) to the parallel worker processes. Because of that, all functions that  you pass to the API must be serializable, as defined by [java.io.Serializable](http://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html).

8. If your  function is an anonymous inner class, consider the following:

9. - Make       the function a standalone class, or a static inner class.
   - Use       a Java 8 lambda function.

10. If your  function is already a static class, check the fields that you assign when you  create an instance of the class. One of the fields most likely holds a  non-serializable type.

11. - In Java, use a RichFunction and       initialize the problematic fields in the open() method.

    - In Scala, you can often simply use “lazy val” to defer initialization until the distributed execution happens.       This may come at a minor performance cost. You can naturally also use a RichFunction in Scala.

      

12. ### Event time无输出的问题

13. Datastream.assignTimestampsAndWatermarks会生成新的stream，所以后续操作应该在新的stream上进行.

14. 

15. ### 异常捕捉

16. http://apache-flink-user-mailing-list-archive.2336050.n4.nabble.com/Fwd-Flink-Exception-Handling-best-practices-tc10034.html>

17. 目前可以使用flink side output能力，把异常数据分流到side output中，不中断主流的处理。



###         event time timezone

​	默认使用UTC，并且目前无法通过tableEnv进行设置。在中国使用flink sql进行处理时，eventtime字段的输出会有8           个小时的时差。 