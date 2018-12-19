# mongodb常用语句

## 1. 数据库操作

### 1.1 创建数据库

- 创建数据库*(如果数据库存在则创建，否则则进入该数据库)*

  `use blog`

  > 注：创建数据库后，表中并不会出现在数据库列表中，需要添加数据后，才会出现在数据库列表中

### 1.2 修改数据库

- 修改数据库的名称

  - 方法1，拷贝当前数据库，再删除旧数据库

    ```js
    db.copyDatabase("blog","blog_new")
    use blog_new
    db.dropDatabase()
    ```

  - 方法2，复制数据库中的集合

    `db.adminCommand({renameCollection:"blog.table1",to:"blog_new.table1"})`


### 1.3 查询数据库

- 查询mongodb中包含的数据库

  `show dbs`

- 查询当前使用的数据库名称

  `db`

### 1.4 删除数据库

- 删除数据库*(删除的是当前正在使用的数据库)*

  `db.dropDatabase`

## 2. 表操作

### 2.1 创建数据库表

- 创建一张数据库表

  `db.createCollection("student")`

### 2.2  修改数据库表名

- 修改数据库的表名称

  `db.student.renameCollection("student_new")`

### 2.3  查询

- 查询数据库中的所有表

  `show collections`

### 2.4 删除

- 删除数据库中的某张表

  `db.students.drop()`

## 3. 数据操作

### 3.1 插入数据

- 插入一条数据

  `db.student.insert({"name":"boolean"})`

- 插入多条数据*(此处插入便于后面进行查询)*

  ```js
  db.student.insert([{
      "_id": "T001",
      "name": "yjt",
      "age": 23,
      "sex": "man",
      "score": {
        "english": 76,
        "chinese": 74,
        "math": 67
      },
      "teacher": [{
          "name": "赵",
          "age": 43,
          "subject": "math"
        },
        {
          "name": "钱",
          "age": 54,
          "subject": "chinese"
        },
        {
          "name": "钱",
          "age": 23,
          "subject": "english"
        }
      ],
      "habit": ["eat", "code"]
    },
    {
      "_id": "T002",
      "name": "yan",
      "age": 24,
      "sex": "woman",
      "score": {
        "english": 66,
        "chinese": 77
      },
      "teacher": [{
          "name": "钱",
          "age": 54,
          "subject": "chinese"
        },
        {
          "name": "钱",
          "age": 23,
          "subject": "english"
        }
      ],
      "habit": ["run", "code"]
    },
    {
      "_id": "T003",
      "name": "tao",
      "age": 28,
      "sex": "man",
      "score": {
        "english": 88
      },
      "teacher": [{
        "name": "钱",
        "age": 23,
        "subject": "english"
      }],
      "habit": ["read", "movie"]
    }
  ])
  ```


### 3.2 更新表中的数据

- 向数组中插入一条数据**(此处向habit中插入一条数据)**

  `db.student.update({"_id":"T003"},{$addToSet:{"habit":"music"}})`

- 删除表中的某一个字段**(此处删除id为T001的habit字段)

  `db.student.update({"_id":"T001"},{$unset:{"habit":""}},false,true)`

- 更新某个字段的值**(此处将id为T001的age更新为20)

  `db.student.update({"_id":"T001"},{$set:{"age":20}})`

- 为某张表添加某个字段**(此处将id为T003的数据添加一个habit字段)**

  `db.student.update({"_id":"T003"},{$set:{habit:["read","music"]}},{multi:true})`

- 添加一个字段（url 代表表名 , 添加字段 content。 字符串类型）

    `db.url.update({}, {$set: {content:""}}, {multi: 1})`

- 删除一个字段

    `db.url.update({},{$unset:{'content':''}},false, true)`

### 3.3  删除表中的数据

- 删除数据库中符合条件的数据**(此处删除name为yjt的数据)**

  `db.student.remove({"name":"yjt"})`

- 删除数据库中所有的数据

  `db.student.remove({})`

### 3.4 查询语句

- 查询一张表的所有数据

  `db.student.find()`

- 查询一张表的数据，按照某个字段排序**(此处按照排序)**

  `db.student.find().sort({"age":1})`

  > 注：1为升序，-1为降序

- 查询一张表的数据，按照条件查询**(此处查询id为T003的数据)**

  `db.student.find({"_id":"T003"})`

- 查询一张表的某个字段

  `db.student.find({},{"_id":1,"name":1,"teacher":1})`

  > 注：find()中第一个参数为查询条件，第二个参数为要查询的字段

- 分组查询**(根据性别分组)**

  ```json
  [{
      "sex": "man",
      "num": 2.0
    },
    {
      "sex": "woman",
      "num": 1.0
    }
  ]
  ```

- 查询表中一个数组包含该数据**(查询habit包含run的数据)**

  `db.student.find({"habit":{$in:["run"]}})`

- 查询文档中一个数组是否包含该数据**(查询teacher的name为钱并且age为23的数据)**

  `db.student.find({ "teacher": { $elemMatch: { "name": "钱", "age": 23} } })`

- 查询某个字段是否存在

  `db.student.find({"habit":{$exists:false}})`

  > true查询该字段存在的数据，false查询该字段不存在的数据

- 模糊查询**(查询name中包含y的数据)**

  `db.student.find({'name':/y/})`





> 本文档持续更新，我会将我遇到的mongdb语句一直放在上面
>
> [MongoDB修改数据库名-其實很簡單](https://blog.csdn.net/chandoudeyuyi/article/details/80038743)
>
> [Runoob-Mongodb教程](http://www.runoob.com/mongodb/mongodb-tutorial.html)