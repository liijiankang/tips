```
1.map过程产生大量对象
2.数据不平衡
3.coalesec调用
4.shuffle后
5.共用对象能够减少OOM的情况
```
```
1.mapPartion代替map,或者连续使用map
2.broadcast join代替join
3.先filter后join
4.partitionBy优化
5.combineByKey使用
6.参数优化
```

```
    shuffle过程，由shufflemanager管理
    shuffle writer 和shuffle reader

```