## python c++ 扩展 -- 类

> 参考 python文档 [newtypes](https://docs.python.org/2/extending/newtypes.html)

提供类比提供方法大概带来以下好处：

* 封装
* 可以在类对象的生命周期结束时添加析构函数
* 在new一个对象时添加构造函数
* 对于需要维护某种resource的情况下（比如mysql链接），选择类是比较好的

### 类型定义

首先需要定义一种新的类型，类似于用C去写一个python class，定义的结构如下

```c++
/**
 * Bplib Object definition.
 */
static PyTypeObject bplib_BplibType = {
    PyObject_HEAD_INIT(NULL)
    0,                         /*ob_size*/
    "bplib.Bplib",             /*tp_name*/
    sizeof(bplib_BplibObject), /*tp_basicsize*/
    0,                         /*tp_itemsize*/
    (destructor)BplibDestructor,          /*tp_dealloc*/
    0,                         /*tp_print*/
    0,                         /*tp_getattr*/
    0,                         /*tp_setattr*/
    0,                         /*tp_compare*/
    0,                         /*tp_repr*/
    0,                         /*tp_as_number*/
    0,                         /*tp_as_sequence*/
    0,                         /*tp_as_mapping*/
    0,                         /*tp_hash */
    0,                         /*tp_call*/
    0,                         /*tp_str*/
    0,                         /*tp_getattro*/
    0,                         /*tp_setattro*/
    0,                         /*tp_as_buffer*/
    Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE, /*tp_flags*/
    "bplib objects",           /* tp_doc */
    0,                     /* tp_traverse */
    0,                     /* tp_clear */
    0,                     /* tp_richcompare */
    0,                     /* tp_weaklistoffset */
    0,                     /* tp_iter */
    0,                     /* tp_iternext */
    Bplib_methods,             /* tp_methods */
    0,                         /* tp_members */
    0,                         /* tp_getset */
    0,                         /* tp_base */
    0,                         /* tp_dict */
    0,                         /* tp_descr_get */
    0,                         /* tp_descr_set */
    0,                         /* tp_dictoffset */
    (initproc)Bplib_init,      /* tp_init */   //FIXME
    0,                         /* tp_alloc */
    0,     //Noddy_new,                 /* tp_new */   //FIXME
};
```

|值| 解释 |
| -- | -- |
|bplib.Bplib| python中的类名 |
|(destructor)BplibDestructor | 析构函数地址 |
| Bplib_methods | 定义member functions的结构体地址 |
| (initproc)Bplib_init| 构造函数地址 |


下面是析构函数的实现：

```c++
/**
 * Destructor for python
 */
static void BplibDestructor(bplib_BplibObject* self) {
    self->subscribe.uninit();
    self->publisher.uninit();
    self->ob_type->tp_free((PyObject*)self);
}
```
这里基本上做了一些释放链接的工作和反初始化的工作，常见的比如mysql链接这样的resource，都需要在这个函数中手动关闭，因为不能保证用户会调用close方法去关闭链接。

```c++
/**
 * ByMethodDef is used to bind a bounch of funcs
 * to a custom python type. Now we just bind 2 funcs
 * to the Bplib object.
 */
static PyMethodDef Bplib_methods[] = {
    {"initSubscriber", (PyCFunction)initSubscriber, METH_VARARGS, "initSubscriber"},
    {"initPublisher", (PyCFunction)initPublisher, METH_VARARGS, "initPublisher"},
    {"uninitSubscriber", (PyCFunction)uninitSubscriber, METH_NOARGS, "uninitPublisher"},
    {"uninitPublisher", (PyCFunction)uninitPublisher, METH_NOARGS, "uninitPublisher"},
    {"get", (PyCFunction)get, METH_VARARGS, "get"},
    {"put", (PyCFunction)put, METH_VARARGS, "put"},
    {NULL}  /* Sentinel */
};
```

每这里定义了6个成员方法，最后一个为占位使用，以第一个为例子：

* `initSubscriber` 为方法名称
* `(PyCFunction)initSubscriber` 为方法地址
* `METH_VARARGS` 表示该方法需要接受参数
* `initSubscriber` doc string

下面是 initSubscriber 方法的实现：

```c++
/**
 * env initialize, will raise an exception when
 * error occured. wrap a try catch to this
 *
 * params in python
 *
 * @param pipename  string
 * @param token     string
 * @param confpath  string
 * @param conffile  string
 *
 * @return 1
 */
static PyObject* initSubscriber(bplib_BplibObject* self, PyObject *args) {
    int _errno, ret;
    char _error[1024];
    char *pipename, *token, *confpath, *conffile;

    PyArg_ParseTuple(args, "ssss", &pipename, &token, &confpath, &conffile);

    com_loadlog(confpath, conffile);
    bigpipe_stomp_t::getinstance()->ignore_bad_pipe();

    ret = self->subscribe.init_env(pipename, token, confpath, conffile);
    if (0 != ret) {
        strcpy(_error, "bigpipe init env failed");
        _errno = ret;
        BpError = PyErr_NewException("bp.error", NULL, NULL);
        PyObject *errorObject = Py_BuildValue("si", _error, _errno);
        PyErr_SetObject(BpError, errorObject);
        return NULL;
    }

    self->is_bp_subscribe_init = 1;
    return Py_BuildValue("i", 0);
}
```


这个方法，跟前一篇说到的一般的方法，最大的不同在于方法的第一个参数，`bplib_BplibObject* self`，这个参数包括之前在析构函数中也有这个参数，这个参数实际上是指向


















