## SparseArray应用场景：
虽说SparseArray性能比较好，但是由于其添加、查找、删除数据都需要先进行一次二分查找，所以在数据量大的情况下性能并不明显，将降低至少50%。

满足下面两个条件我们可以使用SparseArray代替HashMap：

数据量不大，最好在千级以内
key必须为int类型，这中情况下的HashMap可以用SparseArray代替

## ArrayMap应用场景
数据量不大，最好在千级以内
数据结构类型为Map类型

## SparseArray和ArrayMap选择
假设数据量都在千级以内的情况下：

1、如果key的类型已经确定为int类型，那么使用SparseArray，因为它避免了自动装箱的过程，如果key为long类型，它还提供了一个LongSparseArray来确保key为long类型时的使用

2、如果key类型为其它的类型，则使用ArrayMap

## 参考
https://blog.csdn.net/u010687392/article/details/47809295
