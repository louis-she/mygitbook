grep it faster!
===============

工作需求，需要grep一个大约20G的文件，昨天偷懒想直接获得答案就去so上提了问，结果最佳答案是“自己试试就知道了”，于是就自己试试了：（

机器配置：

* cpu 8 cores
* mem 80GB
* ssd disk

听说改locale会让grep更快于是实验前加上了`export LC_ALL=C`
（也做了不改locale的实验，貌似加了这个并没有变快，so上有人说快了两倍，不知道是不是grep版本的原因）

----------------------------

**Input**
```
grep 1351939801 senselist0.log
```
**Output**
```
real	0m12.671s
user	0m9.104s
sys     0m3.538s
```

----------------------------

**Input**
```
begin=$(date +%s); 
time fgrep "1351939801" senselist0.log; 
end=$(date +%s); 
echo "total time: $((end-begin))"```
**Output**
```
real	0m12.594s
user	0m9.146s
sys     0m3.418s
total time: 13
```

----------------------------

**Input**
```
begin=$(date +%s); time parallel -j100% --pipepart --block 100M -a senselist0.log fgrep "1351939801"; end=$(date +%s); echo "total time: $((end-begin))"```
**Output**
```
real	0m3.286s
user	0m10.367s
sys     0m13.615s
total time: 3
```

----------------------------

**Input**
```

```
**Output**
```

```

----------------------------

**Input**
```

```
**Output**
```

```

----------------------------

**Input**
```

```
**Output**
```

```

----------------------------


**Input**
```
time LC_ALL=C parallel --pipe --block 100M grep 1351939801 < senselist0.log 2>/dev/null
```

**Output**

```
real	0m46.055s
user	0m25.843s
sys     1m7.126s
```