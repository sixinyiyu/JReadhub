
`Error`和`Exception`都是继承于`Throwable`，`RuntimeException`及其子类称为未检查异常

* `Error`类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢出等。如`java.lang.StackOverFlowError`和`Java.lang.OutOfMemoryError`。对于这类错误，Java编译器不去检查他们。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和预防，遇到这样的错误，建议让程序终止。

* `Exception`类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常。


常见的异常有

* `NullPointerException`

* `ClassNotFoundException`

* `IllegalArgumentException`

* `NoSuchMethodException`

* `NumberFormatException`

* `IndexOutOfBoundsException`

* `ArrayIndexOutofBoundException`

* `FileNotFoundException`

* `ClassCastException`

* `SQLException`、`IOException`、`ConnectException`