## mongodb查询指定字段

```java
    @Test
    public void fun1() {

        DBObject fieldsObject = new BasicDBObject();
        fieldsObject.put("_id", true);
        fieldsObject.put("name", true);
        fieldsObject.put("code", true);
        fieldsObject.put("marketPrice", true);

        BasicDBObject dbObject = new BasicDBObject();
        fieldsObject.put("productId", "PD329321375342788608");

        Query query = new BasicQuery(dbObject,fieldsObject);

        List<Goods> goodsList = mongoTemplate.find(query, Goods.class);
        String gson = new Gson().toJson(goodsList);
        System.out.println(gson);
    }
```

