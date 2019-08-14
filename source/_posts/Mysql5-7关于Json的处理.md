---
title: Mysql5.7+ 关于Json的处理 - 牢骚向
date: 2019-08-14 00:28:07
tags: [Mysql, Java]
categories: Back-End
---

最近在对数据库处理的时候, 遇到关于 Json 类型. 从 Sql 上来看, json 相关函数的加入对与现有的很多数据结构都是非常友好的
其中用的比较多的函数包裹

```sql
 JSON_SEARCH()
 JSON_EXTRACT()
```

`JSON_SEARCH` 函数用于检索某列下的某个特定 key 值的位置信息.但是不能获得具体的对象
`JSON_EXTRACT` 函数根据具体的 json 索引可以获取整个 json 对象, 例如 JsonArray 中的某个元素

但是两者联合使用的效果, 暂时没有实验出满意的结果

但是, 目前主流框架, 以及哪怕 jdbc 对于 json 的支持并不是很友好.

 <!--more-->

拿最主流的 Mybatis 与 Hibernate 来举例子:

### Mybatis

要在 Mybatis 实现对 Json 的解析, 需要继承 Mybatis 的 `TypeHandler` 接口

```java
public class MyHander<T> extends BaseTypeHandler<T>{

    public MyHandler(Class<T> clazz){
        ...
    }

     @Override
    public void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, this.toJson(parameter));
    }

    @Override
    public T getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return this.toObject(rs.getString(columnName), clazz);
    }

    @Override
    public T getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return this.toObject(rs.getString(columnIndex), clazz);
    }

    @Override
    public T getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return this.toObject(cs.getString(columnIndex), clazz);
    }

    private String toJson(T object) {
        try {
            return mapper.writeValueAsString(object);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private T toObject(String content, Class<?> clazz) {
        if (content != null && !content.isEmpty()) {
            try {
                return (T) mapper.readValue(content, clazz);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        } else {
            return null;
        }
    }

    static {
        mapper.configure(Feature.WRITE_NULL_MAP_VALUES, false);
        mapper.setSerializationInclusion(Inclusion.NON_NULL);
    }
    ...
}
```

实现之后还需要在 Mapper 中像注解或者 xml 配置文件中, 对相对应的字段进行类型描述.

总之就是步骤很麻烦的实现

### Hibernate - dslquery

目前利用 hibernate 也是基于 dslquery 实现的一套框架, 不考虑性能上来说, 对于 json 的查询还是很方便的, 因为只需要

注解声明是 @type(json)之后, 就可以直接存储. 但是确不支持单个的读取.

也就是说, 我只能通过写源生 sql 来查明我所需要数据的列, 然后再把整条列都查出来, 在对其进行一个筛选.

这一点在业务规范和小数据量的时候, 是没有问题的.

至少在代码书写上, 算是最省心的一种了

```java
 @Query(value = "select apply_code from credit_application_status where to_be_processed is not null and to_be_processed ->
  '$[*].handler'  like concat('%', :handler, '%') or to_be_processed -> '$[*].position' like concat('%', :position, '%')", nativeQuery = true)
    List<String> toBeProcessedTaskId(@Param(value = "handler") String username, @Param(value = "position") String position);
```

但是目前, 除了自己实现以外, 没有其它更好的方法来实现对于 json 的高效且省心的操作.

有时候, 还不如直接存入 json 字符串操作来的方便.

`json 数据类型需谨慎使用`
