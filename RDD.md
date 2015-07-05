##RDD（Resilient Distributed Dataset） in Scala


####每个RDD主要由以下五个属性表示
1. 分片集合，每个RDD都可以进行若干个划分，由若干个分片组成
2. 计算函数，一个函数用来计算每个分片的
3. 依赖集合，指出该RDD依赖那些RDD
4. 首选位置（可选），分片P能够被访问的最快的位置，对于HDFS的RDD来说，有三个备份，访问速度可能不同，这里返回最快的那个
5. 划分者（可选），针对key-value的RDD，说明这个RDD是hash划分还是range划分  


####**类：org.apache.spark.rdd.RDD**  
####function:  
**1. def ++(other:RDD[T]):RDD[T]**    

返回当前RDD和指定参数中的RDD的并集，这里并不去重，需要用distinct方法去重    

**2. def aggregate\[U\](zeroValue:U)(seqOp:(U,T)=>u,combOp:(U,U)=>U)(implicit arg0:ClassTag\[U\]):U**    

这个函数看着很复杂，实质和Scala中fold差不多，Scala中fold能做非同质聚合，Spark中aggregate也可以做非同质聚合，下面详细的介绍一下**Scala中的fold、foldLeft、foldRight**和Spark中的**aggregate**  
fold系列函数来自**GenTraversableOnce[A]**这个类，这个类是一个模板特质，对于所有可遍历的对象来说，可以实现并行的遍历。这个特质中的方法要么是抽象方法要么就是可以用该特质中的其他方法实现的。  

**Scala中的聚合函数：** 

- fold方法，abstract def fold\[A1 >: A\](z: A1)(op: (A1, A1) ⇒ A1): A1  
聚合这个可遍历的对象，用指定的操作函数op进行聚合，遍历顺序未指定，可能是非确定的，z为初始值，op为操作符，最终返回聚合计算结果  
val list1=List(1,2,3,4,5)  
list1.fold(0)((a,b)=>a+b)  
结果为15

- foldLeft方法 abstract def foldLeft[B](z: B)(op: (B, A) ⇒ B): B  
可以认为和fold一样，只不过是指定了顺序为从左到右
- foldRight方法 abstract def foldRight[B](z: B)(op: (A, B) ⇒ B): B  
计算顺序从右到左  

**注意：foldLeft和foldRight可能每次运行的结果都不一样，除非这个可遍历的集合是排好序的或者是操作符是可交换的，使用时需要注意初始值在函数参数中的位置**  

**Spark中的aggregrate:**  
def aggregate\[U\](zeroValue:U)(seqOp:(U,T)=>u,combOp:(U,U)=>U)(implicit arg0:ClassTag\[U\]):U  
zeroValue为初始值，T是可遍历集合的类型，U是目标类型，seqOp为操作符，将（U,T）变为U，这里为什么会多了一个combOp呢，因为Spark解决的是并行和分布式问题，每个节点上都生成各自的类型为U的结果，那么如何将这些类型为U的结果合并成一个呢，比如每个U都是一个Map类型，那么理想情况应该根据Map的key合并，所以需要提供合并方法combOp来解决这个问题。  

**3. def cache():RDD.this.type**    

对RDD的数据进行持久化，默认持久化的等级是MEMORY_ONLY  
  
**4. def cartesian\[U\](other:RDD\[U\])(implicit arg0:ClassTag\[U\]):RDD\[(T,U)\]**    

返回当前集合和给定集合的笛卡尔积  

**5.def checkpoint():Unit**    

为这个RDD设置检查点，即校验点，这个RDD将会被存储到校验目录下，这个目录可以在SparkContext.setCheckpointDir()这个方法中设置，一旦该RDD设置为校验点，那么它所依赖的RDD都会被删除了，因为已经不需要在保留父RDD的信息了，相当于数据库中的日志了，有问题或者计算直接从这点开始即可。建议将该RDD持久化到内存中，否则需要重新计算。

**6. def coalesce(numPartitions:Int,shuffle:Boolean=false)(implicit ord:Ordering\[T\]=null):RDD\[T\]**  
   
将当前RDD变为指定数量的Partion的RDD，这个变化可以变大也可变小。  
如原来是有1000个分片，变为100个分片，形成一个narrow依赖，标识变量shuffle表示是否进行shuffle操作，如果极端的coalesce如numPartitions为1，表示变成一个Partition,那么最好用true.一般narrow依赖不需shuffle.