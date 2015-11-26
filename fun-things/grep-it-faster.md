grep it faster!
===============

工作需求，需要grep一个大约20G的文件，就有了下面的实验：




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