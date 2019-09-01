
幂等性

> 一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同



### 数据库锁


> 乐观锁

```sql
update table set xxx1=xxx,xxx2=xxx where id and version = 1
利用版本号
update table set amount=xxx,xxx2=xxx where id and amount>0 
利用临界条件

```


> 悲观锁

```sql
select columns from table  where condition for update

其中condition一定是主键或者唯一索引不然锁表会有问题
```