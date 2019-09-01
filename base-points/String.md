# String、StringBuffer、StringBuilder区别


* `String`、`StringBuffer`、`StringBuilder`是final类不可变，不可继承

* `StringBuffer`线程安全的(方法用`synchronized`修饰)，`StringBuilder`不是线程安全

* `String`每次对字符串进行处理都会创建一个新的`String`对象，不会影响到原有对象，`String`的拼接
方法(+,concat,append)append最快；+每次都会创建`StringBuilder`对象并调用toString，concat每次都会new String(),而append是对字符串本身操作

* `StringBuffer`对其修改的操作不会创建新的字符串对象所以在内存使用方面是优于`String`的，而`StringBuilder`是线程不安全的所以比`StringBuffer更快`