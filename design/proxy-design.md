# 设计模式の代理模式

代理模式，代理模式给某一个对象提供一个代理对象，并由代理对象控制对原对象的引用。 土话的话就是生活中的中介。

生活中我们租房，一般的情况下都是通过中介来租房；先说不通过中介找房的情况，我们先要去搜索到房源信息，然后一个一个沟通约见房东看房，最后找到合适的房子签订合同；那通过中介，我们只需要告诉中介租房诉求，中介找到匹配的房源直接带去看，找到合适的就签订了，整个过程基本出去特殊情况完全不会与房东有其他的接触，整个过程中中介就代理了房东的角色。

#### 代理模式作用

* 首先完全替代了被代理者，拥有与被代理者相同的接口(提供等同的功能)，起到了隔离原代理对象(某些特殊情况下被代理对象不希望被可见)

* 开闭原则，首先代理者与被代理对象拥有相同的接口，被代理者还可以用很多自定义的接口，可以在提供同等代理功能的情况前后进行特定的功能比如日志、缓存等。这样在不修改被代理者的情况可以获得更多额外的功能，符合程序的开闭原则。


Java中共有2种代理模式：静态代理、动态代理

##### 静态代理


##### 动态代理