##aggregate

```
aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U)
```
aggregate函数将每个分区进行seqOp,且从zeroValue开始遍历分区里的所有元素.然后用combOp,从zeroValue开始遍历所有分区的结果.

注意:每个partition的seqOp只应用一次zeroValue,最后的combOp也应用一次zeroValue.

例子:

```
scala> def seq(a:Int,b:Int):Int={
     | println("seq:"+a+":"+b)
     | math.min(a,b)}
seq: (a: Int, b: Int)Int

scala> def comb(a:Int,b:Int):Int={
     | println("comb:"+a+":"+b)
     | a+b}
comb: (a: Int, b: Int)Int

val z =sc.parallelize(List(1,2,4,5,8,9),3)
scala> z.aggregate(3)(seq,comb)
seq:3:4
seq:3:1
seq:1:2
seq:3:8
seq:3:5
seq:3:9
comb:3:1
comb:4:3
comb:7:3
res10: Int = 10
```


----------
##treeAggregate

```
treeAggregate[U: ClassTag](zeroValue: U)(
      seqOp: (U, T) => U,
      combOp: (U, U) => U,
      depth: Int = 2)
```
与aggregate不同的地方是:在每个分区,会做两次或者多次combOp,避免将所有局部的值传给driver端.另外,经过测验初始值zeroValue不会参与combOp.

例子:

```
scala> z.treeAggregate(3)(seq,comb)
seq:3:4
seq:3:5
seq:3:1
seq:1:2
seq:3:8
seq:3:9
comb:3:3
comb:6:1
res12: Int = 7
```
对比图:
![aggregatevstreeaggregate](http://img.blog.csdn.net/20160802160845633)
注释:
Aggregate
 

 1. each executor holds a portion of learning set
 2.  broadcast model to excutors
 3. collect results to driver
 
TreeAggregate

 1. simple heuristic to add level
 2. perform partial aggregation by shipping results to other executors(by repartitioning)
