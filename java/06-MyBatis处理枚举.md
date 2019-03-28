### MyBatis处理枚举类型

MyBatis保存枚举类型的数据存入数据库时，报错，无法存入数据库，因此我们使用MyBatis内置的枚举处理器来处理

#### 1. 枚举类

```java
public enum Delete {
    /**
     * 正常
     */
    NONE(0,"正常"),
    /**
     * 回收站
     */
    TRASH(1,"回收站"),
    /**
     * 缓冲区
     */
    BUFFER(2,"缓冲区");


    private Integer code;
    private String name;

    Delete(Integer code, String name) {
        this.code = code;
        this.name = name;
    }
		//...
}

```



#### 2. 实体类

```java
public class Admin extends BaseEntity {

	private static final long serialVersionUID = 1L;

    /**
     * 用户名
     */
	private String username;

    /**
     * 密码MD5
     */
	private String password;

    /**
     * 姓名
     */
	private String name;

	/**
	 * 是否删除
	 */
	protected Delete delete;
  
  //...
}
```



#### 2. 数据库

```sql
CREATE TABLE `admin` (
	`id` CHAR(20) NOT NULL COMMENT 'id' COLLATE 'utf8_bin',
	`username` VARCHAR(20) NOT NULL COMMENT '用户名' COLLATE 'utf8_bin',
	`password` CHAR(32) NOT NULL COMMENT '密码MD5' COLLATE 'utf8_bin',
	`name` VARCHAR(20) NULL DEFAULT NULL COMMENT '姓名' COLLATE 'utf8_bin',
	`login_failure_count` TINYINT(4) NOT NULL DEFAULT '0' COMMENT '登录失败次数',
	`delete` TINYINT(4) NOT NULL COMMENT '是否删除',
	`create_date` DATETIME NOT NULL COMMENT '创建日期',
	`modify_date` DATETIME NOT NULL COMMENT '修改日期',
	`sort` BIGINT(20) UNSIGNED NOT NULL DEFAULT '0' COMMENT '排序',
	PRIMARY KEY (`id`)
)
COMMENT='管理员'
COLLATE='utf8_bin'
ENGINE=InnoDB
;

```

#### 3. mybatis-EnumTypeHandler处理器

```xml
<resultMap id="BaseResultMap" type="com.onegene.service.official.entity.Admin">
		<id column="id" property="id" jdbcType="VARCHAR" />
		<result column="username" property="username" jdbcType="VARCHAR" />
		<result column="password" property="password" jdbcType="VARCHAR" />
		<result column="name" property="name" jdbcType="VARCHAR" />
		<result column="delete" property="delete" typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
	</resultMap>

<insert id="save" useGeneratedKeys="true" keyProperty="id" parameterType="Admin">
	  INSERT INTO admin
	  (<include refid="columns"/>)
	  VALUES 
	  (
		#{id},
		#{username},
		#{password},
		#{name},
		#{delete,typeHandler=org.apache.ibatis.type.EnumTypeHandler}
	  )
	</insert>

```

#### 4. mybatis-EnumOrdinalTypeHandler处理器

```xml
<resultMap id="BaseResultMap" type="com.onegene.service.official.entity.Admin">
		<id column="id" property="id" jdbcType="VARCHAR" />
		<result column="username" property="username" jdbcType="VARCHAR" />
		<result column="password" property="password" jdbcType="VARCHAR" />
		<result column="name" property="name" jdbcType="VARCHAR" />
		<result column="delete" property="delete" typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
	</resultMap>

<insert id="save" useGeneratedKeys="true" keyProperty="id" parameterType="Admin">
	  INSERT INTO admin
	  (<include refid="columns"/>)
	  VALUES 
	  (
		#{id},
		#{username},
		#{password},
		#{name},
		#{delete,typeHandler=org.apache.ibatis.type.EnumOrdinalTypeHandler}
	  )
	</insert>

```

