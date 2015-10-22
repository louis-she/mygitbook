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

|bplib.Bplib| python中的模块名称 |
| -- | -- |
|  (destructor)BplibDestructor | 析构函数地址 |
| -- | -- |
|   | 析构函数地址 |
| -- | -- |
| -- | -- |
| -- | -- |
| -- | -- |
| -- | -- |



