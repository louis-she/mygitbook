
## python c++ 扩展 -- 函数


### 让扩展提供函数

#### 从入口开始

```c++
static PyMethodDef bpMethods[] = {
    {"read", wrap_bigpipe_subscriber, METH_VARARGS, "read from bigpipe"},
    {"write", wrap_bigpipe_publisher, METH_VARARGS, "write to bigpipe"},
    {NULL, NULL}
};


#ifndef PyMODINIT_FUNC
#define PyMODINIT_FUNC void
#endif
PyMODINIT_FUNC
initbplib(void) {
    PyObject* m;

    bplib_BplibType.tp_new = PyType_GenericNew;
    if (PyType_Ready(&bplib_BplibType) < 0)
        return;

    m = Py_InitModule("bplib", bpMethods);
    Py_INCREF(&bplib_BplibType);
    PyModule_AddObject(m, "Bplib", (PyObject *)&bplib_BplibType);
}
```

本段内容基本可以复制粘贴，大概是在import的时候会调用的，主要关注
```c++
m = Py_InitModule("bplib", bpMethods);
```

第一个参数为扩展名称，在DEBUG的时候可能会用到，第二个参数为上面定义的struct结构，该结构重点关注一下：

```c++
static PyMethodDef bpMethods[] = {
    {"read", wrap_bigpipe_subscriber, METH_VARARGS, "read from bigpipe"},
    {"write", wrap_bigpipe_publisher, METH_VARARGS, "write to bigpipe"},
    {NULL, NULL}
};
```

这个结构有三行，第三行的`{NULL, NULL}` 为占位用的，前两行则是我们定义的方法，其中
```c++
{"read", wrap_bigpipe_subscriber, METH_VARARGS, "read from bigpipe"}
```
* `read` 表示方法名，这个名称则是python中的函数名
* `wrap_bigpipe_subscriber` 为方法在cpp代码中的名称
* `METH_VARARGS` 表示该函数接受参数
* `read from bigpipe` 函数的doc string，python中可以看到

这样python扩展就知道他需要向外提供的方法的所有信息了，不过具体的实现还没有写，就是`wrap_bigpipe_subscriber` 和 `wrap_bigpipe_publisher` 这两个C++函数

#### C++ function 2 Python function

首先，提供的函数都需要在上面的入口中注册，这里就以`wrap_bigpipe_subscriber` 为例子

```c++
PyObject* wrap_bigpipe_subscriber(PyObject* self, PyObject* args) {
    int _errno, pipe_id, msg_id, index = 0;
    char _error[1024];
    char *pipename, *token, *confpath, *conffile;
    vector<bigpipe_recv_msg_t> precv_node_list;
    bigpipe_epoll_event_t epoll_node;
    PyObject *result, *tmpresult;

    if (! PyArg_ParseTuple(args, "llssss", &msg_id, &pipe_id, &pipename, &token, &confpath, &conffile))
        return NULL;

    precv_node_list = read_from_bigpipe(msg_id, pipe_id, pipename, token, confpath, conffile, _error, &_errno);
    if (precv_node_list.size() == 0) {
        BpError = PyErr_NewException("bp.error", NULL, NULL);
        PyObject *errorObject = Py_BuildValue("si", _error, _errno);
        PyErr_SetObject(BpError, errorObject);
        return NULL;
    }

    result = PyList_New(precv_node_list.size());
    for (vector<bigpipe_recv_msg_t>::iterator it = precv_node_list.begin();
        it != precv_node_list.end(); ++it) {
        tmpresult = Py_BuildValue("s#", it->msg,
            it->len);
        PyList_SetItem(result, index, tmpresult);
        index++;
    }

    return result;
}
```

先看参数： **PyObject\* self, PyObject\* args**

第一个参数必不可少，self应该是来标记模块本身的
第二个参数，如果该函数接收参数的话（在入口处定义），那么这里第二个参数也是必不可少的

接下来是最重要的
```c++
if (! PyArg_ParseTuple(args, "llssss", &msg_id, 
    &pipe_id, &pipename, &token, &confpath, &conffile))
```
这一行代码

**PyArg_ParseTuple** 是用来将python **格式化** 到指定的C++变量中的， 比如这里 format 为 "llssss" 则表示该python方法接收6个参数，从左至右的类型依次为long,long,string,string,string,string,string，会依次写入到后面传入的6个C变量中（如果变量类型不一致可能会core dump），具体的format可以取的字符串可以参考 [这里](https://docs.python.org/2/c-api/arg.html)

然后需要关注

```c++
result = PyList_New(precv_node_list.size());
//...
PyList_SetItem(result, index, tmpresult);
```

**PyList_New** 宏实际上会创建一个PyObject对象，注意在python中，所以的类型实际上都是PyObject类型的，包括int，string等，这个宏返回的PyObject在python中实际上是一个List对象，这里可以用`PyList_SetItem`去为这个PyObject对象增加元素。

```c++
tmpresult = Py_BuildValue("s#", it->msg, it->len);
```

**Py_BuildValue** 也是一个经常使用的宏，该宏可以创建任意一个python对象，这里使用这个宏创建了一个**二进制**对象，具体的使用方法见 [这里](https://docs.python.org/2/c-api/arg.html)

**返回值**

返回值必须为PyObject对象，C++中的返回值会直接返回给python调用者


### 编译

```c
gcc -Ixxx/include/python2.7     \\
    -Ixxx/lib/python2.7/config  \\
    -lxxx/libpython2.7.a        \\
    -shared 
```






