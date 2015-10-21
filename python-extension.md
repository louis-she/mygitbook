

## python c++ 扩展



### 让扩展提供函数

首先从入口开始

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








