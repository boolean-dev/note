## redis的基本操作

### String

| 命令 | 行为           | 示例        |
| ---- | -------------- | ----------- |
| GET  | 获取字符串的值 | GET name zs |
| SET  | 设置字符串的值 | SET name zs |
| DEL  | 删除           | DEL name    |

### List

| 命令   | 行为                         | 示例             |
| ------ | ---------------------------- | ---------------- |
| RPUSH  | 将值推入到列表右端           | RPUSH list name  |
| LRANGE | 获取列表范围内的值           | RRANGE list 0 -1 |
| LINDEX | 获取列表指定元素上的单个元素 | RANGE list 1     |
| LPOP   | 从左端弹出一个元素           | LPOP list        |

