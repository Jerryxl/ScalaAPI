##RDD（Resilient Distributed Dataset） in Scala


####每个RDD主要由以下五个属性表示
1. 分片集合，每个RDD都可以进行若干个划分，由若干个分片组成
2. 计算函数，一个函数用来计算每个分片的
3. 依赖集合，指出该RDD依赖那些RDD
4. 首选位置（可选），分片P能够被访问的最快的位置，对于HDFS的RDD来说，有三个备份，访问速度可能不同，这里返回最快的那个
5. 划分者（可选），针对key-value的RDD，说明这个RDD是hash划分还是range划分  


####**类：org.apache.spark.rdd.RDD**  
####function:  
1. def ++(other:RDD[T]):RDD[T]  
返回当前RDD和指定参数中的RDD的并集，这里并不去重，需要用distinct方法去重
2. aggregate\[U\](zeroValue:U)(seqOp:(U,T)=>u,combOp:(U,U)=>U)(implicit arg0:ClassTag\[U\]):U  
这个函数看着很复杂，实质和fold差不多，但fold只能做同质聚合，aggregate则可以做非同质聚合，下面详细的介绍一下fold、foldLeft、foldRight和aggregate  
