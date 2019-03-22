### gson无法正常将时间戳转化成date



#### gson将时间戳转化成date时，报错

`Failed to parse date ["1551950239757']: Invalid time zone indicator '3'`

#### 解决办法

添加一个long转date的解析器

```java
		@Test
    public void fun1(){

        GsonBuilder builder = new GsonBuilder();

        // Register an adapter to manage the date types as long values
        builder.registerTypeAdapter(Date.class, new JsonDeserializer<Date>() {
            public Date deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
                return new Date(json.getAsJsonPrimitive().getAsLong());
            }
        });

        Gson gson = builder.create();


        String str = "{\"name\":\"yjt\",\"date\":\"1552012460277\"}";
        Person person = gson.fromJson(str,Person.class);
        log.info("{}", person);
    }
```
文章来自[stackoverflow](https://stackoverflow.com/questions/5671373/unparseable-date-1302828677828-trying-to-deserialize-with-gson-a-millisecond)
