背景：使用数据库存储 `json` 格式数据时，需要通过 `MyBatis` 转换映射为自定义数据类型。

## 一、项目依赖

```xml
<dependencies>
    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.10</version>
    </dependency>
    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.26</version>
    </dependency>
    <!-- Jackson 用于 JSON 处理 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.13.0</version>
    </dependency>
</dependencies>
```

## 二、编写自己的TypeHandler

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedJdbcTypes;
import org.apache.ibatis.type.MappedTypes;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

@MappedJdbcTypes(JdbcType.VARCHAR)  // 处理 JDBC VARCHAR 类型字段
@MappedTypes({List.class})          // 映射到 Java 的 List 类型
public class StringListTypeHandler extends BaseTypeHandler<List<String>> {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    //------------------- 序列化：Java → 数据库 -------------------
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i,
                                    List<String> parameter, JdbcType jdbcType) throws SQLException {
        try {
            String json = objectMapper.writeValueAsString(parameter);
            ps.setString(i, json);
        } catch (JsonProcessingException e) {
            throw new SQLException("JSON 序列化失败: " + e.getMessage(), e);
        }
    }

    //------------------- 反序列化：数据库 → Java -------------------
    @Override
    public List<String> getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return parseJson(rs.getString(columnName));
    }

    @Override
    public List<String> getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return parseJson(rs.getString(columnIndex));
    }

    @Override
    public List<String> getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return parseJson(cs.getString(columnIndex));
    }

    // 统一 JSON 解析逻辑
    private List<String> parseJson(String json) {
        if (json == null || json.isEmpty()) {
            return null;
        }
        try {
            return objectMapper.readValue(json, new TypeReference<List<String>>() {
            });
        } catch (Exception e) {
            throw new RuntimeException("JSON 反序列化失败: " + e.getMessage(), e);
        }
    }
}
```

## 三、注册typeHandler

注册多个不同的包时使用 **逗号** 分割

```xml
mybatis:
	type-handlers-package: 自定义typeHandler包路径
```

## 四、Mapper XML中配置属性typeHandler

```xml
<resultMap id="BaseResultMap" type="cn.probiecoder.Task">
    <result column="tasks" property="tasks" jdbcType="VARCHAR"  typeHandler="cn.probiecoder.config.StringListTypeHandler"/>
</resultMap>
```

使用`MyBatis`进行对象映射时，经过以上就可以实现

**注意事项：**

当使用 `MyBatis-Plus` 等框架时，如果同一个 `List` 需要进行不同类型的转换包装，请注意：在使用 `MyBatis-Plus` 提供的 `API`（如 `selectByPrimaryKey`）时，字段映射会默认使用第一个匹配的 `typeHandler`。因此，需要在实体字段上添加注解来指定所需的 `typeHandler`。

```java
@ColumnType(typeHandler = StringListTypeHandler.class)
```

## 五、typeHandler支持范型

### 1、 抽象父类，实现具体的类型转换逻辑

```java
public abstract class JsonListTypeHandler<T> extends TypeReference<T> implements TypeHandler<List<T>> {}

public abstract class JsonTypeHandler<T> extends TypeReference<T> implements TypeHandler<T> {}
```

反序列化类型转换：

```java
// List集合
JSON.parseArray(json, (Class<T>) getRawType());

// 普通对象
JSON.parseObject(json, this.getRawType());
```

### 2、子类继承父类并增加类型转换注解

```java
@MappedJdbcTypes(JdbcType.VARCHAR)  // 处理 JDBC VARCHAR 类型字段
@MappedTypes({List.class})          // 映射到 Java 的 List 类型
public class SubChildTypeHandler extends JsonListTypeHandler<String> {}

@MappedJdbcTypes(JdbcType.VARCHAR)
@MappedTypes(InstanceExtInfoVo.class)
public class SubChildTypeHandler extends JsonTypeHandler<String> {
```