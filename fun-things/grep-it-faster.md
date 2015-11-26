grep it faster!
===============

工作需求，需要grep一个大约20G的文件，就有了下面的实验：

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
time LC_ALL=C grep 1351939801 senselist0.log
```
**Output**
```
real	0m12.594s
user	0m9.146s
sys     0m3.418s
```

----------------------------

**Input**
```
time LC_ALL=C fgrep 1351939801 senselist0.log
```
**Output**
```
real	0m12.632s
user	0m9.137s
sys     0m3.462s
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