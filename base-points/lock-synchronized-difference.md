#### ```synchronized```

```synchronized```关键字加锁是jvm底层的实现，主要是基于```mori```指令，锁的释放是由jvm自己释放的，不需要手动释放锁，一旦某个线程进入了```synchronized```修饰的代码块就会立即获取锁，并执行其中代码而其他线程则等待释放锁。所以```synchronized```是非公平锁！

