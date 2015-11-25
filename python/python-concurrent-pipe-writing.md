python 多进程写pipe
===================

项目中进程间通信用到了 `multiprocessing.pipe`, 本来是想用`multiprocessing.Queue`来做的，不过在使用之后发现Queue内部貌似是信号量实现的，而由于环境的原因，`sem_open`调用总是失败，所以没办法只有上pipe了。

然后后来遇到了多进程同时往一个pipe中写，单一work进程去消费的场景，在man了pipe之后发现，pipe的写入是根据 **system page size**去写的，linux对pipe的描述可查[man page](http://man7.org/linux/man-pages/man7/pipe.7.html)，不过由于python的pipe是封装过后的，所以做了如下实验进行了相关验证：

```python
import os
import sys
import time
from multiprocessing import Pipe

# generate pipe pair
parent_conn, child_conn = Pipe()
# 1MB
size = 1024 * 1024

for i in range(0, 2):
    ret = os.fork()
    if ret == 0:
        # child process send bytes to parent process repeatly
        while True:
            recv = child_conn.send_bytes("process %s sent %s\n" % (i, str(i) * size))
            time.sleep(0.1)
        sys.exit(0)

# parent process receive bytes sent from child process
while True:
    data = parent_conn.recv_bytes()
    with open("/tmp/received", "a") as received_file:
        received_file.write(data)
```

这里使用了`send_bytes` 而没用 `send` 也是由于场景需求，在原程序里的通信数据为protobuf的二进制数据

程序很简单，两个子进程，写同一个pipe，父进程读这个pipe：

测试后发现：

* 当size设置为1024的时候，程序的行为跟预期一样
* 当size设置为1024 * 1024的时候，最后的结果就乱套了

这里说明还是不能多个进程写一个pipe的
