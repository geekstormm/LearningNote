# STL知识

## vector

#### vector 内存用尽后,是怎么扩容的？

STL的扩容因子是2，vector容量满了以后直接申请多一倍的容量，这样存在的问题是每次新申请的内存都比之前的内存要多，对缓存不友好，所以folly和RapidJSON的扩容因子都是1.5，因为这样扩容几次后可以复用之前的空间。

