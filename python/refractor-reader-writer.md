重构reader writer
=================

* 风格按照[goodcoder](https://github.com/louis-sherren/goodcoder)评审的标准
* 抛弃所有含义不明确的名称，例如**helper**, **common**, **core**, 命名不累赘，比如`user.get_user_name` 实际上改为`user.get_name`
* 多使用higher-order function