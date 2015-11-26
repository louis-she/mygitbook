grep it faster!
===============

工作需求，需要grep一个大约20G的文件，昨天偷懒想直接获得答案就去so上提了问，结果最佳答案是“自己试试就知道了”，于是就自己试试了：（

环境和配置：

* cpu 8 cores
* mem 80GB
* ssd disk
* 测试文件18G 23717441行

听说改locale会让grep更快于是实验前加上了`export LC_ALL=C`，原因是

> Simple byte comparison vs multiple byte character comparison

但在不改locale的情况也做了实验，貌似改之后并没有变快，so上有人说快了两倍，不知道是不是grep版本的原因

----------------------------

#### 无脑grep

**Input**
```
begin=$(date +%s); 
time grep "1351939801" senselist0.log; 
end=$(date +%s); 
echo "total time: $((end-begin))"
```
**Output**
```
real	0m12.624s
user	0m9.178s
sys     0m3.417s
total time: 12
```

----------------------------

#### 无脑fgrep

**Input**
```
begin=$(date +%s); 
time fgrep "1351939801" senselist0.log; 
end=$(date +%s); 
echo "total time: $((end-begin))"
```
**Output**
```
real	0m12.607s
user	0m9.102s
sys     0m3.477s
total time: 12
```

跟grep比起来没什么不同

----------------------------

#### AWK也试试

**Input**
```
begin=$(date +%s); time parallel -j100% --pipepart --block 100M -a senselist0.log fgrep "1351939801"; end=$(date +%s); echo "total time: $((end-begin))"```
**Output**
```
real	0m35.802s
user	0m31.264s
sys     0m4.463s
total time: 36
```

不能忍

----------------------------

**Input**
```
begin=$(date +%s); 
time awk '{if($3==8842431714) {print $0}}' senselist0.log; 
end=$(date +%s); 
echo "total time: $((end-begin))"
```
**Output**
```
real	0m35.866s
user	0m31.278s
sys     0m4.515s
total time: 36
```

用户CPU时间太长了

----------------------------

#### parallel 文件分100M，并发数默认为cpu核心数

**Input**
```shell
begin=$(date +%s); 
time parallel --pipepart --block 100M -a senselist0.log -k grep "8842431714"; 
end=$(date +%s); 
echo "total time: $((end-begin))"
```
**Output**
```
real	0m3.195s
user	0m10.020s
sys     0m13.591s
total time: 4
```
用了parallel后总体时间比之前快多了

----------------------------

#### parallel 文件500MB，并发数默认

**Input**
```
begin=$(date +%s); 
time parallel --pipepart --block 500M -a senselist0.log -k grep "8842431714"; 
end=$(date +%s); 
echo "total time: $((end-begin))"
```
**Output**
```
real	0m3.485s
user	0m9.527s
sys     0m12.772s
total time: 3
```

比刚刚还快了一些

----------------------------


**Input**
```
begin=$(date +%s); 
time parallel --pipepart --block 500M -P0 -a senselist0.log -k grep "8842431714"; 
end=$(date +%s); 
echo "total time: $((end-begin))"
```

**Output**

```
real	0m3.154s
user	0m9.765s
sys     0m13.141s
total time: 3
```

-P0表示尽可能得多并发，看来已经不能更快了

----------------------------
