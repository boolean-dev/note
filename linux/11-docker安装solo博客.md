# docker安装solo博客

### 脚本

```sh
docker pull b3log/solo
```

### 本地

```sh
docker run --restart=on-failure:10 \
		--detach --name solo --network=host \
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="root123456" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://192.168.12.231:3306/blog?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=80 --server_scheme=http --server_host=192.168.12.188 --server_port=
```

### 正式

```sh
docker run --detach --restart=on-failure:10 --name solo --network=host \ 
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="root123456" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://127.0.0.1:3306/blog?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=8081 --server_scheme=https --server_host=blog.booleandev.xyz --server_port= --restart=on-failure:10
```

```sh
docker run --detach --name solo --restart=on-failure:10 --network=host \
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="root123456" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://127.0.0.1:3306/blog?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=8081 --server_scheme=https --server_host=blog.booleandev.xyz  --server_port= 
```



1.删除b3_solo_archivedate两个字段archiveDateArticleCount，archiveDatePublishedArticleCount

2.删除b3_solo_article三个字段articleHadBeenPublished，articleIsPublished，articleEditorType，添加articleStatus字段，int(11),默认值0

3.删除b3_solo_comment两个字段commentEmail，commentOnType

4.删除b3_solo_page五个字段，pageCommentCount，pageContent，pageCommentable，pageType，pageEditorType

5.删除b3_solo_tag两个字段，tagPublishedRefCount，tagReferenceCount

6.删除b3_solo_user两个字段，userEmail，userPassword