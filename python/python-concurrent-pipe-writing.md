python 多进程写pipe
===================

项目中进程间通信用到了 `multiprocessing.pipe`, 本来是想用`multiprocessing.Queue`来做的，不过在使用之后发现Queue内部貌似是信号量实现的，而由于环境的原因，`sem_open`调用总是失败，所以没办法只有上pipe了。

然后后来遇到了多进程同时往一个pipe中写，单一work进程去消费的场景，在man了pipe之后发现，pipe的写入是根据 **system page size**去写的，linux对pipe的描述可查[man page](http://man7.org/linux/man-pages/man7/pipe.7.html)，不过由于python的pipe是封装过后的，所以做了如下实验进行了相关验证：

```python
#!/bin/python

import os
import sys
import time
from multiprocessing import Pipe

# generate pipe pair
parent_conn, child_conn = Pipe()
count = 0
# 1MB
size = 1024 * 1024

for i in range(0, 2):
    ret = os.fork()
    if ret > 0:
        # child process
        if i == 0:
            while True:
                recv = child_conn.recv_bytes()
                open("/tmp/pipe_test/%s" % count, "w").write(recv)
        
        if i == 1:
            while True:
                if count == 1:
                    break
                parent_conn.send_bytes("1" * size)
                count += 1
            sys.exit(0)

while True:
    if count == 1:
        break
    parent_conn.send_bytes("%" * size)
    count += 1
```

这里使用了`send_bytes` 而没用 `send` 也是由于场景需求，在原程序里的通信数据为protobuf的二进制数据

程序很简单，两个子进程，第一个子进程作为消费方读pipe，第二个子进程作为生产方向pipe中发数据，消费方将读到的数据写入文件，每读取一次写一个新的文件。最后的结果很诡异：

* **结果1** **/tmp/pipe_test/** 中只有一个文件 **0**，并且文件只有1MB，这是第一个有问题的地方，由于是两个进程，每个进程都写了1MB，最后结果应该2MB才对，为什么只有1MB了呢
* **结果2** 0文件中，发现是一段连续的"%"，然后一段连续的"1"，如此反复，我复制了一串连续的"1"和"%"到不同的文件，发现这些文件大小（即连续的1和%的大小）恰好为16K 



更有趣的是，如果把`size`改为一个比较小的数字，比如100，然后再执行程序，最后的结果就是跟期望的一样了， 即：
/tmp/pipe_test/ 下有两个文件： 0 和 1，每个文件有100字节，然后内容也跟预期的一样。

首先可以认定，pipe还是一对一得去读取比较好。其他的，[不懂就问](http://stackoverflow.com/questions/33729216/multiprocess-write-to-one-pipe-in-python)。

