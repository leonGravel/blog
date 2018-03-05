---
title: mybatisXML配置以及XML映射文件
tags: mybatis,xml配置
author : 李鳌
grammar_cjkRuby: true
---
最近在公司内部技术交流会上分享了mybatis相关的配置资料，现在整理下弄到博客上面。<!--more-->
## <a name="XML_"></a>XML 映射配置文件

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置（settings）和属性（properties）信息。文档的顶层结构如下：

*   configuration 配置
    *   [properties 属性](#properties)
    *   [settings 设置](#settings)
    *   [typeAliases 类型别名](#typeAliases)
    *   [typeHandlers 类型处理器](#typeHandlers)
    *   [objectFactory 对象工厂](#objectFactory)
    *   [plugins 插件](#plugins)
    *   [environments 环境](#environments)
        *   environment 环境变量
            *   transactionManager 事务管理器
            *   dataSource 数据源
    *   [databaseIdProvider 数据库厂商标识](#databaseIdProvider)
    *   [mappers 映射器](#mappers)

<a name="properties"></a>

### <a name="properties"></a>properties

这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。例如：

```
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

其中的属性就可以在整个配置文件中使用来替换需要动态配置的属性值。比如:

```
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

这个例子中的 username 和 password 将会由 properties 元素中设置的相应值来替换。 driver 和 url 属性将会由 config.properties 文件中对应的值来替换。这样就为配置提供了诸多灵活选择。

属性也可以被传递到 SqlSessionFactoryBuilder.build()方法中。例如：

```
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, props);

// ... or ...

SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, props);
```

如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：

*   在 properties 元素体内指定的属性首先被读取。
*   然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
*   最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。

因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性。

从MyBatis 3.4.2开始，你可以为占位符指定一个默认值。例如：

```
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${username:ut_user}"/> <!-- If 'username' property not present, username become 'ut_user' -->
</dataSource>
```

这个特性默认是关闭的。如果你想为占位符指定一个默认值， 你应该添加一个指定的属性来开启这个特性。例如：

```
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- Enable this feature -->
</properties>
```

提示： 你可以使用 <tt>":"</tt> 作为属性键(e.g. <tt>db:username</tt>) 或者你也可以在sql定义中使用 OGNL 表达式的三元运算符(e.g. <tt>${tableName != null ? tableName : 'global_constants'}</tt>)， 你应该通过增加一个指定的属性来改变分隔键和默认值的字符。例如：

```
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- Change default value of separator -->
</properties>
```

```
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${db:username?:ut_user}"/>
</dataSource>
```

<a name="settings"></a>

### <a name="settings"></a>settings

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。下表描述了设置中各项的意图、默认值等。


| 设置参数 | 描述 | 有效值 | 默认值 |
| --- | --- | --- | --- |
| cacheEnabled | 该配置影响的所有映射器中配置的缓存的全局开关。 | true | false | true |
| lazyLoadingEnabled | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。特定关联关系中可通过设置<tt>fetchType</tt>属性来覆盖该项的开关状态。 | true | false | false |
| aggressiveLazyLoading | 当启用时，带有延迟加载属性的对象的加载与否完全取决于对任意延迟属性的调用；反之，每种属性将会按需加载。 | true | false | true |
| multipleResultSetsEnabled | 是否允许单一语句返回多结果集（需要兼容驱动）。 | true | false | true |
| useColumnLabel | 使用列标签代替列名。不同的驱动在这方面会有不同的表现，具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。 | true | false | true |
| useGeneratedKeys | 允许 JDBC 支持自动生成主键，需要驱动兼容。如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby）。 | true | false | False |
| autoMappingBehavior | 指定 MyBatis 是否以及如何自动映射指定的列到字段或属性。NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。FULL 会自动映射任意复杂的结果集（包括嵌套和其他情况）。 | NONE, PARTIAL, FULL | PARTIAL |
| defaultExecutorType | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）；BATCH 执行器将重用语句并执行批量更新。 | SIMPLE REUSE BATCH | SIMPLE |
| defaultStatementTimeout | 设置超时时间，它决定驱动等待数据库响应的秒数。 | Any positive integer | Not Set (null) |
| safeRowBoundsEnabled | 允许在嵌套语句中使用行分界（RowBounds）。 | true | false | False |
| mapUnderscoreToCamelCase | 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。 | true | false | False |
| localCacheScope | MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。 | SESSION | STATEMENT | SESSION |
| jdbcTypeForNull | 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType enumeration. Most common are: NULL, VARCHAR and OTHER | OTHER |
| lazyLoadTriggerMethods | 指定哪个对象的方法触发一次延迟加载。 | A method name list separated by commas | equals,clone,hashCode,toString |
| defaultScriptingLanguage | 指定动态 SQL 生成的默认语言。 | A type alias or fully qualified class name. | org.apache.ibatis.scripting.xmltags.XMLDynamicLanguageDriver |
| callSettersOnNulls | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这对于有 Map.keySet() 依赖或 null 值初始化的时候是有用的。注意原始类型（int、boolean等）是不能设置成 null 的。 | true | false | false |
| logPrefix | 指定 MyBatis 增加到日志名称的前缀。 | Any String | Not set |
| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 | SLF4J | LOG4J | LOG4J2 | JDK_LOGGING | COMMONS_LOGGING | STDOUT_LOGGING | NO_LOGGING | Not set |
| proxyFactory | 为 Mybatis 用来创建具有延迟加载能力的对象设置代理工具。 | CGLIB | JAVASSIST | CGLIB |

一个配置完整的 settings 元素的示例如下：

```
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

<a name="typeAliases"></a>

### <a name="typeAliases"></a>typeAliases

类型别名是为 Java 类型设置一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。例如:

```
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

当这样配置时，<tt>Blog</tt>可以用在任何使用<tt>domain.blog.Blog</tt>的地方。

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如:

```
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

每一个在包 <tt>domain.blog</tt> 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 <tt>domain.blog.Author</tt> 的别名为 <tt>author</tt>；若有注解，则别名为其注解值。看下面的例子：

```
@Alias("author")
public class Author {
    ...
}
```

已经为许多常见的 Java 类型内建了相应的类型别名。它们都是大小写不敏感的，需要注意的是由基本类型名称重复导致的特殊处理。

| 别名 | 映射的类型 |
| --- | --- |
| _byte | byte |
| _long | long |
| _short | short |
| _int | int |
| _integer | int |
| _double | double |
| _float | float |
| _boolean | boolean |
| string | String |
| byte | Byte |
| long | Long |
| short | Short |
| int | Integer |
| integer | Integer |
| double | Double |
| float | Float |
| boolean | Boolean |
| date | Date |
| decimal | BigDecimal |
| bigdecimal | BigDecimal |
| object | Object |
| map | Map |
| hashmap | HashMap |
| list | List |
| arraylist | ArrayList |
| collection | Collection |
| iterator | Iterator |

<a name="typeHandlers"></a>

### <a name="typeHandlers"></a>typeHandlers

类型处理器的作用就是

*   查询时把数据库存储的值转换成java类型
*   修改是把java类型转换成数据库类型存储，处理
*   下面这个表格描述了默认的类型处理器。

| 类型处理器 | Java 类型 | JDBC 类型 |
| --- | --- | --- |
| <tt>BooleanTypeHandler</tt> | <tt>java.lang.Boolean</tt>, <tt>boolean</tt> | 数据库兼容的 <tt>BOOLEAN</tt> |
| <tt>ByteTypeHandler</tt> | <tt>java.lang.Byte</tt>, <tt>byte</tt> | 数据库兼容的 <tt>NUMERIC</tt> 或 <tt>BYTE</tt> |
| <tt>ShortTypeHandler</tt> | <tt>java.lang.Short</tt>, <tt>short</tt> | 数据库兼容的 <tt>NUMERIC</tt> 或 <tt>SHORT INTEGER</tt> |
| <tt>IntegerTypeHandler</tt> | <tt>java.lang.Integer</tt>, <tt>int</tt> | 数据库兼容的 <tt>NUMERIC</tt> 或 <tt>INTEGER</tt> |
| <tt>LongTypeHandler</tt> | <tt>java.lang.Long</tt>, <tt>long</tt> | 数据库兼容的 <tt>NUMERIC</tt> 或 <tt>LONG INTEGER</tt> |
| <tt>FloatTypeHandler</tt> | <tt>java.lang.Float</tt>, <tt>float</tt> | 数据库兼容的 <tt>NUMERIC</tt> 或 <tt>FLOAT</tt> |
| <tt>DoubleTypeHandler</tt> | <tt>java.lang.Double</tt>, <tt>double</tt> | 数据库兼容的 <tt>NUMERIC</tt> 或 <tt>DOUBLE</tt> |
| <tt>BigDecimalTypeHandler</tt> | <tt>java.math.BigDecimal</tt> | 数据库兼容的 <tt>NUMERIC</tt> 或 <tt>DECIMAL</tt> |
| <tt>StringTypeHandler</tt> | <tt>java.lang.String</tt> | <tt>CHAR</tt>, <tt>VARCHAR</tt> |
| <tt>ClobReaderTypeHandler</tt> | <tt>java.io.Reader</tt> | - |
| <tt>ClobTypeHandler</tt> | <tt>java.lang.String</tt> | <tt>CLOB</tt>, <tt>LONGVARCHAR</tt> |
| <tt>NStringTypeHandler</tt> | <tt>java.lang.String</tt> | <tt>NVARCHAR</tt>, <tt>NCHAR</tt> |
| <tt>NClobTypeHandler</tt> | <tt>java.lang.String</tt> | <tt>NCLOB</tt> |
| <tt>BlobInputStreamTypeHandler</tt> | <tt>java.io.InputStream</tt> | - |
| <tt>ByteArrayTypeHandler</tt> | <tt>byte[]</tt> | 数据库兼容的字节流类型 |
| <tt>BlobTypeHandler</tt> | <tt>byte[]</tt> | <tt>BLOB</tt>, <tt>LONGVARBINARY</tt> |
| <tt>DateTypeHandler</tt> | <tt>java.util.Date</tt> | <tt>TIMESTAMP</tt> |
| <tt>DateOnlyTypeHandler</tt> | <tt>java.util.Date</tt> | <tt>DATE</tt> |
| <tt>TimeOnlyTypeHandler</tt> | <tt>java.util.Date</tt> | <tt>TIME</tt> |
| <tt>SqlTimestampTypeHandler</tt> | <tt>java.sql.Timestamp</tt> | <tt>TIMESTAMP</tt> |
| <tt>SqlDateTypeHandler</tt> | <tt>java.sql.Date</tt> | <tt>DATE</tt> |
| <tt>SqlTimeTypeHandler</tt> | <tt>java.sql.Time</tt> | <tt>TIME</tt> |
| <tt>ObjectTypeHandler</tt> | Any | <tt>OTHER</tt> 或未指定类型 |
| <tt>EnumTypeHandler</tt> | Enumeration Type | VARCHAR-任何兼容的字符串类型，存储枚举的名称（而不是索引） |
| <tt>EnumOrdinalTypeHandler</tt> | Enumeration Type | 任何兼容的 <tt>NUMERIC</tt> 或 <tt>DOUBLE</tt> 类型，存储枚举的索引（而不是名称）。 |
| <tt>InstantTypeHandler</tt> | <tt>java.time.Instant</tt> | <tt>TIMESTAMP</tt> |
| <tt>LocalDateTimeTypeHandler</tt> | <tt>java.time.LocalDateTime</tt> | <tt>TIMESTAMP</tt> |
| <tt>LocalDateTypeHandler</tt> | <tt>java.time.LocalDate</tt> | <tt>DATE</tt> |
| <tt>LocalTimeTypeHandler</tt> | <tt>java.time.LocalTime</tt> | <tt>TIME</tt> |
| <tt>OffsetDateTimeTypeHandler</tt> | <tt>java.time.OffsetDateTime</tt> | <tt>TIMESTAMP</tt> |
| <tt>OffsetTimeTypeHandler</tt> | <tt>java.time.OffsetTime</tt> | <tt>TIME</tt> |
| <tt>ZonedDateTimeTypeHandler</tt> | <tt>java.time.ZonedDateTime</tt> | <tt>TIMESTAMP</tt> |
| <tt>YearTypeHandler</tt> | <tt>java.time.Year</tt> | <tt>INTEGER</tt> |
| <tt>MonthTypeHandler</tt> | <tt>java.time.Month</tt> | <tt>INTEGER</tt> |
| <tt>YearMonthTypeHandler</tt> | <tt>java.time.YearMonth</tt> | <tt>VARCHAR</tt> or <tt>LONGVARCHAR</tt> |
| <tt>JapaneseDateTypeHandler</tt> | <tt>java.time.chrono.JapaneseDate</tt> | <tt>DATE</tt> |

你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 具体做法为：实现 <tt>org.apache.ibatis.type.TypeHandler</tt> 接口， 或继承一个很便利的类 <tt>org.apache.ibatis.type.BaseTypeHandler</tt>， 然后可以选择性地将它映射到一个 JDBC 类型。比如：

```
// ExampleTypeHandler.java
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```

```
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

使用这个的类型处理器将会覆盖已经存在的处理 Java 的 String 类型属性和 VARCHAR 参数及结果的类型处理器。 要注意 MyBatis 不会窥探数据库元信息来决定使用哪种类型，所以你必须在参数和结果映射中指明那是 VARCHAR 类型的字段， 以使其能够绑定到正确的类型处理器上。 这是因为：MyBatis 直到语句被执行才清楚数据类型。

通过类型处理器的泛型，MyBatis 可以得知该类型处理器处理的 Java 类型，不过这种行为可以通过两种方法改变：

*   在类型处理器的配置元素（typeHandler element）上增加一个 <tt>javaType</tt> 属性（比如：<tt>javaType="String"</tt>）；
*   在类型处理器的类上（TypeHandler class）增加一个 <tt>@MappedTypes</tt> 注解来指定与其关联的 Java 类型列表。 如果在 <tt>javaType</tt> 属性中也同时指定，则注解方式将被忽略。

可以通过两种方式来指定被关联的 JDBC 类型：

*   在类型处理器的配置元素上增加一个 <tt>jdbcType</tt> 属性（比如：<tt>jdbcType="VARCHAR"</tt>）；
*   在类型处理器的类上（TypeHandler class）增加一个 <tt>@MappedJdbcTypes</tt> 注解来指定与其关联的 JDBC 类型列表。 如果在 <tt>jdbcType</tt> 属性中也同时指定，则注解方式将被忽略。

当决定在<tt>ResultMap</tt>中使用某一TypeHandler时，此时java类型是已知的（从结果类型中获得），但是JDBC类型是未知的。 因此Mybatis使用<tt>javaType=[TheJavaType], jdbcType=null</tt>的组合来选择一个TypeHandler。 这意味着使用<tt>@MappedJdbcTypes</tt>注解可以_限制_TypeHandler的范围，同时除非显示的设置，否则TypeHandler在<tt>ResultMap</tt>中将是无效的。 如果希望在<tt>ResultMap</tt>中使用TypeHandler，那么设置<tt>@MappedJdbcTypes</tt>注解的<tt>includeNullJdbcType=true</tt>即可。 然而从Mybatis 3.4.0开始，如果**只有一个**注册的TypeHandler来处理Java类型，那么它将是<tt>ResultMap</tt>使用Java类型时的默认值（即使没有<tt>includeNullJdbcType=true</tt>）。

最后，可以让 MyBatis 为你查找类型处理器：

```
<!-- mybatis-config.xml -->
<typeHandlers>
  <package name="org.mybatis.example"/>
</typeHandlers>
```

注意在使用自动检索（autodiscovery）功能的时候，只能通过注解方式来指定 JDBC 的类型。

你能创建一个泛型类型处理器，它可以处理多于一个类。为达到此目的， 需要增加一个接收该类作为参数的构造器，这样在构造一个类型处理器的时候 MyBatis 就会传入一个具体的类。

```
//GenericTypeHandler.java
public class GenericTypeHandler<E extends MyObject> extends BaseTypeHandler<E> {

  private Class<E> type;

  public GenericTypeHandler(Class<E> type) {
    if (type == null) throw new IllegalArgumentException("Type argument cannot be null");
    this.type = type;
  }
  ...
```

<tt>EnumTypeHandler</tt> 和 <tt>EnumOrdinalTypeHandler</tt> 都是泛型类型处理器（generic TypeHandlers）， 我们将会在接下来的部分详细探讨。

### <a name="a"></a>处理枚举类型

若想映射枚举类型 <tt>Enum</tt>，则需要从 <tt>EnumTypeHandler</tt> 或者 <tt>EnumOrdinalTypeHandler</tt> 中选一个来使用。

比如说我们想存储取近似值时用到的舍入模式。默认情况下，MyBatis 会利用 <tt>EnumTypeHandler</tt> 来把 <tt>Enum</tt> 值转换成对应的名字。

**注意 <tt>EnumTypeHandler</tt> 在某种意义上来说是比较特别的，其他的处理器只针对某个特定的类，而它不同，它会处理任意继承了 <tt>Enum</tt> 的类。**

不过，我们可能不想存储名字，相反我们的 DBA 会坚持使用整形值代码。那也一样轻而易举： 在配置文件中把 <tt>EnumOrdinalTypeHandler</tt> 加到 <tt>typeHandlers</tt> 中即可， 这样每个 <tt>RoundingMode</tt> 将通过他们的序数值来映射成对应的整形。

```
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="java.math.RoundingMode"/>
</typeHandlers>
```

但是怎样能将同样的 <tt>Enum</tt> 既映射成字符串又映射成整形呢？

自动映射器（auto-mapper）会自动地选用 <tt>EnumOrdinalTypeHandler</tt> 来处理， 所以如果我们想用普通的 <tt>EnumTypeHandler</tt>，就非要为那些 SQL 语句显式地设置要用到的类型处理器不可。



```
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.apache.ibatis.submitted.rounding.Mapper">
	<resultMap type="org.apache.ibatis.submitted.rounding.User" id="usermap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<result column="funkyNumber" property="funkyNumber"/>
		<result column="roundingMode" property="roundingMode"/>
	</resultMap>

	<select id="getUser" resultMap="usermap">
		select * from users
	</select>
	<insert id="insert">
	    insert into users (id, name, funkyNumber, roundingMode) values (
	    	#{id}, #{name}, #{funkyNumber}, #{roundingMode}
	    )
	</insert>

	<resultMap type="org.apache.ibatis.submitted.rounding.User" id="usermap2">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<result column="funkyNumber" property="funkyNumber"/>
		<result column="roundingMode" property="roundingMode" typeHandler="org.apache.ibatis.type.EnumTypeHandler"/>
	</resultMap>
	<select id="getUser2" resultMap="usermap2">
		select * from users2
	</select>
	<insert id="insert2">
	    insert into users2 (id, name, funkyNumber, roundingMode) values (
	    	#{id}, #{name}, #{funkyNumber}, #{roundingMode, typeHandler=org.apache.ibatis.type.EnumTypeHandler}
	    )
	</insert>

</mapper>
```

注意，这里的 select 语句强制使用 <tt>resultMap</tt> 来代替 <tt>resultType</tt>。

<a name="objectFactory"></a>

### <a name="aobjectFactory"></a>对象工厂（objectFactory）

MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。 如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。比如：

```
// ExampleObjectFactory.java
public class ExampleObjectFactory extends DefaultObjectFactory {
  public Object create(Class type) {
    return super.create(type);
  }
  public Object create(Class type, List<Class> constructorArgTypes, List<Object> constructorArgs) {
    return super.create(type, constructorArgTypes, constructorArgs);
  }
  public void setProperties(Properties properties) {
    super.setProperties(properties);
  }
  public <T> boolean isCollection(Class<T> type) {
    return Collection.class.isAssignableFrom(type);
  }}
```

```
<!-- mybatis-config.xml -->
<objectFactory type="org.mybatis.example.ExampleObjectFactory">
  <property name="someProperty" value="100"/>
</objectFactory>
```

ObjectFactory 接口很简单，它包含两个创建用的方法，一个是处理默认构造方法的，另外一个是处理带参数的构造方法的。 最后，setProperties 方法可以被用来配置 ObjectFactory，在初始化你的 ObjectFactory 实例后， objectFactory 元素体中定义的属性会被传递给 setProperties 方法。

<a name="plugins"></a>

### <a name="aplugins"></a>插件（plugins）

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

*   Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
*   ParameterHandler (getParameterObject, setParameters)
*   ResultSetHandler (handleResultSets, handleOutputParameters)
*   StatementHandler (prepare, parameterize, batch, update, query)

这些类中方法的细节可以通过查看每个方法的签名来发现，或者直接查看 MyBatis 的发行包中的源代码。 假设你想做的不仅仅是监控方法的调用，那么你应该很好的了解正在重写的方法的行为。 因为如果在试图修改或重写已有方法的行为的时候，你很可能在破坏 MyBatis 的核心模块。 这些都是更低层的类和方法，所以使用插件的时候要特别当心。

通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定了想要拦截的方法签名即可。

```
@Intercepts({ @Signature(type = StatementHandler.class, method = "prepare", args = { Connection.class, Integer.class}) })
public class SQLStatsInterceptor implements Interceptor  {

	private final Logger logger = Logger.getLogger(SQLStatsInterceptor.class);

    @Override
    public Object intercept(Invocation invocation) throws Throwable {

        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
        BoundSql boundSql = statementHandler.getBoundSql();
        String sql = boundSql.getSql();
        logger.info("mybatis intercept sql:{"+sql+"}");
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        String dialect = properties.getProperty("dialect");
        logger.info("mybatis intercept dialect:{"+dialect+"}");
    }

}
```

```
<!-- mybatis-config.xml -->
<plugins>
		<plugin interceptor="com.gravel.plugins.SQLStatsInterceptor">
			<property name="dialect" value="mysql" />
		</plugin>
	</plugins>
```

上面的插件将会拦截在 Executor 实例中所有的 “update” 方法调用， 这里的 Executor 是负责执行低层映射语句的内部对象。

NOTE **覆盖配置类**

除了用插件来修改 MyBatis 核心行为之外，还可以通过完全覆盖配置类来达到目的。只需继承后覆盖其中的每个方法，再把它传递到 SqlSessionFactoryBuilder.build(myConfig) 方法即可。再次重申，这可能会严重影响 MyBatis 的行为，务请慎之又慎。

<a name="environments"></a>

### <a name="aenvironments"></a>配置环境（environments）

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者共享相同 Schema 的多个生产数据库， 想使用相同的 SQL 映射。许多类似的用例。

**不过要记住：尽管可以配置多个环境，每个 SqlSessionFactory 实例只能选择其一。**

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

*   **每个数据库对应一个 SqlSessionFactory 实例**

为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。可以接受环境配置的两个方法签名是：

```
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment,properties);
```

如果忽略了环境参数，那么默认环境将会被加载，如下所示：

```
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader,properties);
```

环境元素定义了如何配置环境。

```
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

注意这里的关键点:

*   默认的环境 ID（比如:default=”development”）。
*   每个 environment 元素定义的环境 ID（比如:id=”development”）。
*   事务管理器的配置（比如:type=”JDBC”）。
*   数据源的配置（比如:type=”POOLED”）。

默认的环境和环境 ID 是一目了然的。随你怎么命名，只要保证默认环境要匹配其中一个环境ID。

**事务管理器（transactionManager）**

在 MyBatis 中有两种类型的事务管理器（也就是 type=”[JDBC|MANAGED]”）：

*   JDBC – 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。
*   MANAGED – 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如:

    ```
    <transactionManager type="MANAGED">
      <property name="closeConnection" value="false"/>
    </transactionManager>
    ```

NOTE如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器， 因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

这两种事务管理器类型都不需要任何属性。它们不过是类型别名，换句话说，你可以使用 TransactionFactory 接口的实现类的完全限定名或类型别名代替它们。

```
public interface TransactionFactory {
  void setProperties(Properties props);  
  Transaction newTransaction(Connection conn);
  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit);  
}
```

任何在 XML 中配置的属性在实例化之后将会被传递给 setProperties() 方法。你也需要创建一个 Transaction 接口的实现类，这个接口也很简单：

```
public interface Transaction {
  Connection getConnection() throws SQLException;
  void commit() throws SQLException;
  void rollback() throws SQLException;
  void close() throws SQLException;
  Integer getTimeout() throws SQLException;
}
```

使用这两个接口，你可以完全自定义 MyBatis 对事务的处理。

**数据源（dataSource）**

dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

*   许多 MyBatis 的应用程序将会按示例中的例子来配置数据源。然而它并不是必须的。要知道为了方便使用延迟加载，数据源才是必须的。

有三种内建的数据源类型（也就是 type=”[UNPOOLED|POOLED|JNDI]”）：

**UNPOOLED**– 这个数据源的实现只是每次被请求时打开和关闭连接。虽然一点慢，它对在及时可用连接方面没有性能要求的简单应用程序是一个很好的选择。 不同的数据库在这方面表现也是不一样的，所以对某些数据库来说使用连接池并不重要，这个配置也是理想的。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

*   <tt>driver</tt> – 这是 JDBC 驱动的 Java 类的完全限定名（并不是JDBC驱动中可能包含的数据源类）。
*   <tt>url</tt> – 这是数据库的 JDBC URL 地址。
*   <tt>username</tt> – 登录数据库的用户名。
*   <tt>password</tt> – 登录数据库的密码。
*   <tt>defaultTransactionIsolationLevel</tt> – 默认的连接事务隔离级别。

作为可选项，你也可以传递属性给数据库驱动。要这样做，属性的前缀为“driver.”，例如：

*   <tt>driver.encoding=UTF8</tt>

这将通过DriverManager.getConnection(url,driverProperties)方法传递值为 <tt>UTF8</tt> 的 <tt>encoding</tt> 属性给数据库驱动。

**POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这是一种使得并发 Web 应用快速响应请求的流行处理方式。

除了上述提到 UNPOOLED 下的属性外，会有更多属性用来配置 POOLED 的数据源：

*   <tt>poolMaximumActiveConnections</tt> – 在任意时间可以存在的活动（也就是正在使用）连接数量，默认值：10
*   <tt>poolMaximumIdleConnections</tt> – 任意时间可能存在的空闲连接数。
*   <tt>poolMaximumCheckoutTime</tt> – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
*   <tt>poolTimeToWait</tt> – 这是一个底层设置，如果获取连接花费的相当长的时间，它会给连接池打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直安静的失败），默认值：20000 毫秒（即 20 秒）。
*   <tt>poolMaximumLocalBadConnectionTolerance</tt> – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程. 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这 个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 <tt>poolMaximumIdleConnections</tt> 与 <tt>poolMaximumLocalBadConnectionTolerance</tt> 之和。 默认值：3 (Since: 3.4.5)
*   <tt>poolPingQuery</tt> – 发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。
*   <tt>poolPingEnabled</tt> – 是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 <tt>poolPingQuery</tt> 属性（最好是一个非常快的 SQL），默认值：false。
*   <tt>poolPingConnectionsNotUsedFor</tt> – 配置 poolPingQuery 的使用频度。这可以被设置成匹配具体的数据库连接超时时间，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

**JNDI**– 这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。这种数据源配置只需要两个属性：

*   <tt>initial_context</tt> – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么 data_source 属性将会直接从 InitialContext 中寻找。
*   <tt>data_source</tt> – 这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给初始上下文。比如：

*   <tt>env.encoding=UTF8</tt>

这就会在初始上下文（InitialContext）实例化时往它的构造方法传递值为 <tt>UTF8</tt> 的 <tt>encoding</tt> 属性。

通过需要实现接口 <tt>org.apache.ibatis.datasource.DataSourceFactory</tt> ， 也可使用任何第三方数据源，：

```
public interface DataSourceFactory {
  void setProperties(Properties props);
  DataSource getDataSource();
}
```

<tt>org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory</tt> 可被用作父类来构建新的数据源适配器，比如下面这段插入 C3P0 数据源所必需的代码：

```
import org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory;
import com.mchange.v2.c3p0.ComboPooledDataSource;

public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {

  public C3P0DataSourceFactory() {
    this.dataSource = new ComboPooledDataSource();
  }
}
```

为了令其工作，为每个需要 MyBatis 调用的 setter 方法中增加一个属性。下面是一个可以连接至 PostgreSQL 数据库的例子：

```
<dataSource type="org.myproject.C3P0DataSourceFactory">
  <property name="driver" value="org.postgresql.Driver"/>
  <property name="url" value="jdbc:postgresql:mydb"/>
  <property name="username" value="postgres"/>
  <property name="password" value="root"/>
</dataSource>
```

<a name="databaseIdProvider"></a>

### <a name="databaseIdProvider"></a>databaseIdProvider

MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 <tt>databaseId</tt> 属性。 MyBatis 会加载不带 <tt>databaseId</tt> 属性和带有匹配当前数据库 <tt>databaseId</tt> 属性的所有语句。 如果同时找到带有 <tt>databaseId</tt> 和不带 <tt>databaseId</tt> 的相同语句，则后者会被舍弃。 为支持多厂商特性只要像下面这样在 mybatis-config.xml 文件中加入 <tt>databaseIdProvider</tt> 即可：

```
<databaseIdProvider type="DB_VENDOR" />
```

这里的 DB_VENDOR 会通过 <tt>DatabaseMetaData#getDatabaseProductName()</tt> 返回的字符串进行设置。 由于通常情况下这个字符串都非常长而且相同产品的不同版本会返回不同的值，所以最好通过设置属性别名来使其变短，如下：

```
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver"/>
  <property name="DB2" value="db2"/>        
  <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

在有 properties 时，DB_VENDOR databaseIdProvider 的将被设置为第一个能匹配数据库产品名称的属性键对应的值，如果没有匹配的属性将会设置为 “null”。 在这个例子中，如果 <tt>getDatabaseProductName()</tt> 返回“Oracle (DataDirect)”，databaseId 将被设置为“oracle”。

你可以通过实现接口 <tt>org.apache.ibatis.mapping.DatabaseIdProvider</tt> 并在 mybatis-config.xml 中注册来构建自己的 DatabaseIdProvider：

```
public interface DatabaseIdProvider {
  void setProperties(Properties p);
  String getDatabaseId(DataSource dataSource) throws SQLException;
}
```

<a name="mappers"></a>

### <a name="amappers"></a>映射器（mappers）

既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要定义 SQL 映射语句了。但是首先我们需要告诉 MyBatis 到哪里去找到这些语句。 Java 在自动查找这方面没有提供一个很好的方法，所以最佳的方式是告诉 MyBatis 到哪里去找映射文件。你可以使用相对于类路径的资源引用， 或完全限定资源定位符（包括 <tt>file:///</tt> 的 URL），或类名和包名等。例如：

```
<!-- Using classpath relative resources -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

```
<!-- Using url fully qualified paths -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
```

```
<!-- Using mapper interface classes -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

```
<!-- Register all interfaces in a package as mappers -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

这些配置会告诉了 MyBatis 去哪里找映射文件.

## <a name="Mapper_XML_"></a>Mapper XML 文件

MyBatis 的真正强大在于它的映射语句，也是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 就是针对 SQL 构建的，并且比普通的方法做的更好。

SQL 映射文件有很少的几个顶级元素（按照它们应该被定义的顺序）：

*   <tt>cache</tt> – 给定命名空间的缓存配置。
*   <tt>cache-ref</tt> – 其他命名空间缓存配置的引用。
*   <tt>resultMap</tt> – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
*   <tt>~~parameterMap~~</tt> ~~– 已废弃！老式风格的参数映射。内联参数是首选,这个元素可能在将来被移除，这里不会记录。~~
*   <tt>sql</tt> – 可被其他语句引用的可重用语句块。
*   <tt>insert</tt> – 映射插入语句
*   <tt>update</tt> – 映射更新语句
*   <tt>delete</tt> – 映射删除语句
*   <tt>select</tt> – 映射查询语句

下一部分将从语句本身开始来描述每个元素的细节。

<a name="select"></a>

### <a name="select"></a>select

查询语句是 MyBatis 中最常用的元素之一，光能把数据存到数据库中价值并不大，如果还能重新取出来才有用，多数应用也都是查询比修改要频繁。对每个插入、更新或删除操作，通常对应多个查询操作。这是 MyBatis 的基本原则之一，也是将焦点和努力放到查询和结果映射的原因。简单查询的 select 元素是非常简单的。比如：

```
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

这个语句被称作 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。

注意参数符号：

```
#{id}
```

这就告诉 MyBatis 创建一个预处理语句参数，通过 JDBC，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：

```
// Similar JDBC code, NOT MyBatis…
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

当然，这需要很多单独的 JDBC 的代码来提取结果并将它们映射到对象实例中，这就是 MyBatis 节省你时间的地方。我们需要深入了解参数和结果映射，细节部分我们下面来了解。

select 元素有很多属性允许你配置，来决定每条语句的作用细节。

```
<select
        <!--  1. id （必须配置）
        id是命名空间中的唯一标识符，可被用来代表这条语句。 
        一个命名空间（namespace） 对应一个dao接口, 
        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致  -->
     
     id="selectPerson"
     
     <!-- 2. parameterType （可选配置, 默认为mybatis自动选择处理）
        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->
     parameterType="int"
     
     <!-- 3. resultType (resultType 与 resultMap 二选一配置)
         resultType用以指定返回类型，指定的类型可以是基本类型，可以是java容器，也可以是javabean -->
     resultType="hashmap"
     
     <!-- 4. resultMap (resultType 与 resultMap 二选一配置)
         resultMap用于引用我们通过 resultMap标签定义的映射类型，这也是mybatis组件高级复杂映射的关键 -->
     resultMap="personResultMap"
     
     <!-- 5. flushCache (可选配置)
         将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false -->
     flushCache="false"
     
     <!-- 6. useCache (可选配置)
         将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true -->
     useCache="true"
     
     <!-- 7. timeout (可选配置) 
         这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）-->
     timeout="10000"
     
     <!-- 8. fetchSize (可选配置) 
         这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动)-->
     fetchSize="256"
     
     <!-- 9. statementType (可选配置) 
         STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED-->
     statementType="PREPARED"
     
     <!-- 10. resultSetType (可选配置) 
         FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）-->
     resultSetType="FORWARD_ONLY">
```
配置看起来总是这么多，不过实际常用的配置也就那么几个， 根据自己的需要，上面都已注明是否必须配置。

<a name="insert_update_and_delete"></a>

### <a name="insert_update__delete"></a>insert, update 和 delete

数据变更语句 insert，update 和 delete 的实现非常接近：

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

<!-- mapper 为根元素节点， 一个namespace对应一个dao -->
<mapper namespace="com.dy.dao.UserDao">

    <insert
      <!-- 1. id （必须配置）
        id是命名空间中的唯一标识符，可被用来代表这条语句。 
        一个命名空间（namespace） 对应一个dao接口, 
        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致 -->
      
      id="insertUser"
      
      <!-- 2. parameterType （可选配置, 默认为mybatis自动选择处理）
        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->
      
      parameterType="com.demo.User"
      
      <!-- 3. flushCache （可选配置，默认配置为true）
        将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句） -->
      
      flushCache="true"
      
      <!-- 4. statementType （可选配置，默认配置为PREPARED）
        STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 -->
      
      statementType="PREPARED"
      
      <!-- 5. keyProperty (可选配置， 默认为unset)
        （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
      
      keyProperty=""
      
      <!-- 6. keyColumn     (可选配置)
        （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
      
      keyColumn=""
      
      <!-- 7. useGeneratedKeys (可选配置， 默认为false)
        （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。  -->
      
      useGeneratedKeys="false"
      
      <!-- 8. timeout  (可选配置， 默认为unset, 依赖驱动)
        这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 -->
      timeout="20">

    <update
      id="updateUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">

    <delete
      id="deleteUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">
</mapper>
```

下面就是 insert，update 和 delete 语句的示例：

```
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>

<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```

如前所述，插入语句的配置规则更加丰富，在插入语句里面有一些额外的属性和子元素用来处理主键的生成，而且有多种生成方式。

首先，如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上就OK了。例如，如果上面的 Author 表已经对 id 使用了自动生成的列类型，那么语句可以修改为:

```
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

如果你的数据库还支持多行插入, 你也可以传入一个<tt>Author</tt>s数组或集合，并返回自动生成的主键。

```
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>
```

对于不支持自动生成类型的数据库或可能不支持自动生成主键 JDBC 驱动来说，MyBatis 有另外一种方法来生成主键。

这里有一个简单（甚至很傻）的示例，它可以生成一个随机 ID（你最好不要这么做，但这里展示了 MyBatis 处理问题的灵活性及其所关心的广度）：

```
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```
同理，如果我们在使用mysql的时候，想在数据插入后返回插入的id, 我们也可以使用 selectKey 这个元素：
```
<!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
           <!-- oracle等不支持id自增长的，可根据其id生成策略，先获取id 
           
        <selectKey resultType="int" order="BEFORE" keyProperty="id">
              select seq_user_id.nextval as id from dual
        </selectKey>
        
        --> 
        
        <!-- mysql插入数据后，获取id -->
        <selectKey keyProperty="id" resultType="int" order="AFTER" >
               SELECT LAST_INSERT_ID() as id
           </selectKey>
          
           insert into user(id, name, age) 
               values(#{id}, #{name} #{age})
   </insert>
```

在上面的示例中，selectKey 元素将会首先运行，Author 的 id 会被设置，然后插入语句会被调用。这给你了一个和数据库中来处理自动生成的主键类似的行为，避免了使 Java 代码变得复杂。

selectKey 元素描述如下：

```
<selectKey
        <!-- selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
        keyProperty="id"
        <!-- 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 -->
        resultType="int"
        <!-- 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 -->
        order="BEFORE"
        <!-- 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 -->
        statementType="PREPARED">
```



### <a name="aParameters"></a>参数（Parameters）

前面的所有语句中你所见到的都是简单参数的例子，实际上参数是 MyBatis 非常强大的元素，对于简单的做法，大概 90% 的情况参数都很少，比如：

```
<select id="selectUsers" resultType="User">
  select id, username, password
  from users
  where id = #{id}
</select>
```

上面的这个示例说明了一个非常简单的命名参数映射。参数类型被设置为 <tt>int</tt>，这样这个参数就可以被设置成任何内容。原生的类型或简单数据类型（比如整型和字符串）因为没有相关属性，它会完全用参数值来替代。然而，如果传入一个复杂的对象，行为就会有一点不同了。比如：

```
<insert id="insertUser" parameterType="User">
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>
```

如果 User 类型的参数对象传递到了语句中，id、username 和 password 属性将会被查找，然后将它们的值传入预处理语句的参数中。

这点对于向语句中传参是比较好的而且又简单，不过参数映射的功能远不止于此。

首先，像 MyBatis 的其他部分一样，参数也可以指定一个特殊的数据类型。

```
#{property,javaType=int,jdbcType=NUMERIC}
```

像 MyBatis 的剩余部分一样，javaType 通常可以从参数对象中来去确定，前提是只要对象不是一个 HashMap。那么 javaType 应该被确定来保证使用正确类型处理器。

NOTE 如果 null 被当作值来传递，对于所有可能为空的列，JDBC Type 是需要的。你可以自己通过阅读预处理语句的 setNull() 方法的 JavaDocs 文档来研究这种情况。

为了以后定制类型处理方式，你也可以指定一个特殊的类型处理器类（或别名），比如：

```
#{age,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler}
```

尽管看起来配置变得越来越繁琐，但实际上是很少去设置它们。

对于数值类型，还有一个小数保留位数的设置，来确定小数点后保留的位数。

```
#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}
```

最后，mode 属性允许你指定 IN，OUT 或 INOUT 参数。如果参数为 OUT 或 INOUT，参数对象属性的真实值将会被改变，就像你在获取输出参数时所期望的那样。如果 mode 为 OUT（或 INOUT），而且 jdbcType 为 CURSOR(也就是 Oracle 的 REFCURSOR)，你必须指定一个 resultMap 来映射结果集到参数类型。要注意这里的 javaType 属性是可选的，如果左边的空白是 jdbcType 的 CURSOR 类型，它会自动地被设置为结果集。

```
#{department, mode=OUT, jdbcType=CURSOR, javaType=ResultSet, resultMap=departmentResultMap}
```

MyBatis 也支持很多高级的数据类型，比如结构体，但是当注册 out 参数时你必须告诉它语句类型名称。比如（再次提示，在实际中要像这样不能换行）：

```
#{middleInitial, mode=OUT, jdbcType=STRUCT, jdbcTypeName=MY_TYPE, resultMap=departmentResultMap}
```

尽管所有这些强大的选项很多时候你只简单指定属性名，其他的事情 MyBatis 会自己去推断，最多你需要为可能为空的列名指定 <tt>jdbcType</tt>。

```
#{firstName}
#{middleInitial,jdbcType=VARCHAR}
#{lastName}
```

#### <a name="a"></a>字符串替换

默认情况下,使用#{}格式的语法会导致 MyBatis 创建预处理语句属性并安全地设置值（比如?）。这样做更安全，更迅速，通常也是首选做法，不过有时你只是想直接在 SQL 语句中插入一个不改变的字符串。比如，像 ORDER BY，你可以这样来使用：

```
ORDER BY ${columnName}
```

这里 MyBatis 不会修改或转义字符串。

NOTE 以这种方式接受从用户输出的内容并提供给语句中不变的字符串是不安全的，会导致潜在的 SQL 注入攻击，因此要么不允许用户输入这些字段，要么自行转义并检验。

<a name="Result_Maps"></a>

### <a name="Result_Maps"></a>Result Maps

resultMap 元素是 MyBatis 中最重要最强大的元素。它就是让你远离 90%的需要从结果 集中取出数据的 JDBC 代码的那个东西, 而且在一些情形下允许你做一些 JDBC 不支持的事 情。 事实上, 编写相似于对复杂语句联合映射这些等同的代码, 也许可以跨过上千行的代码。 ResultMap 的设计就是简单语句不需要明确的结果映射,而很多复杂语句确实需要描述它们 的关系。

你已经看到简单映射语句的示例了,但没有明确的 resultMap。比如:

```
<select id="selectUsers" resultType="map">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

这样一个语句简单作用于所有列被自动映射到 HashMap 的键上,这由 resultType 属性 指定。这在很多情况下是有用的,但是 HashMap 不能很好描述一个领域模型。那样你的应 用程序将会使用 JavaBeans 或 POJOs(Plain Old Java Objects,普通 Java 对象)来作为领域 模型。MyBatis 对两者都支持。看看下面这个 JavaBean:

```
package com.someapp.model;
public class User {
  private int id;
  private String username;
  private String hashedPassword;

  public int getId() {
    return id;
  }
  public void setId(int id) {
    this.id = id;
  }
  public String getUsername() {
    return username;
  }
  public void setUsername(String username) {
    this.username = username;
  }
  public String getHashedPassword() {
    return hashedPassword;
  }
  public void setHashedPassword(String hashedPassword) {
    this.hashedPassword = hashedPassword;
  }
}
```

基于 JavaBean 的规范,上面这个类有 3 个属性:id,username 和 hashedPassword。这些 在 select 语句中会精确匹配到列名。

这样的一个 JavaBean 可以被映射到结果集,就像映射到 HashMap 一样简单。

```
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

要记住类型别名是你的伙伴。使用它们你可以不用输入类的全路径。比如:

```
<!-- In mybatis-config.xml file -->
<typeAlias type="com.someapp.model.User" alias="User"/>

<!-- In SQL Mapping XML file -->
<select id="selectUsers" resultType="User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

这些情况下,MyBatis 会在幕后自动创建一个 ResultMap,基于属性名来映射列到 JavaBean 的属性上。如果列名没有精确匹配,你可以在列名上使用 select 字句的别名(一个 基本的 SQL 特性)来匹配标签。比如:

```
<select id="selectUsers" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```

ResultMap 最优秀的地方你已经了解了很多了,但是你还没有真正的看到一个。这些简 单的示例不需要比你看到的更多东西。 只是出于示例的原因, 让我们来看看最后一个示例中 外部的 resultMap 是什么样子的,这也是解决列名不匹配的另外一种方式。

```
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
```

引用它的语句使用 resultMap 属性就行了(注意我们去掉了 resultType 属性)。比如:

```
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

如果世界总是这么简单就好了。

#### <a name="a"></a>高级结果映射

MyBatis 创建的一个想法:数据库不用永远是你想要的或需要它们是什么样的。而我们 最喜欢的数据库最好是第三范式或 BCNF 模式,但它们有时不是。如果可能有一个单独的 数据库映射,所有应用程序都可以使用它,这是非常好的,但有时也不是。结果映射就是 MyBatis 提供处理这个问题的答案。


1.首先，我们先看看一个常见的博客页面的组成，如下：

a.页面上能够展示的部分：正文，标题，日期，作者，评论正文，评论时间，评论人等等

b.页面之外的部分：用户名，用户id，用户密码，用户基本信息（电话，邮箱，地址，兴趣，特长，等等）

2.将我们页面上的信息从数据库中查出来的SQL语句转化为Mapper文件中的语句，可能是如下内容：


```
  <select id="selectBlogDetails" resultMap="detailedBlogResultMap">  
  select  
  B.id as blog_id,  
  B.title as blog_title,  
  B.author_id as blog_author_id,  
  A.id as author_id,  
  A.username as author_username,  
 A.password as author_password,  
  A.email as author_email,  
 A.bio as author_bio,  
  A.favourite_section as author_favourite_section,  
  P.id as post_id,  
  P.blog_id as post_blog_id,  
  P.author_id as post_author_id,  
  P.created_on as post_created_on,  
  P.section as post_section,  
  P.subject as post_subject,  
  P.draft as draft,  
  P.body as post_body,  
  C.id as comment_id,  
 C.post_id as comment_post_id,  
  C.name as comment_name,  
  C.comment as comment_text,  
  T.id as tag_id,  
 T.name as tag_name  
 from Blog B  
 left outer join Author A on B.author_id = A.id  
  left outer join Post P on B.id = P.blog_id  
 left outer join Comment C on P.id = C.post_id  
 left outer join Post_Tag PT on PT.post_id = P.id  
  left outer join Tag T on PT.tag_id = T.id  
  where B.id = #{id}  
 </select>  
```
其对应着非常复杂的结果集合，Mapper文件可能长这个样子，如下：
```

 <!-- Very Complex Result Map -->  
  <resultMap id="detailedBlogResultMap" type="Blog">  
  <constructor>  
  <idArg column="blog_id" javaType="int"/>  
 </constructor>  
  <result property="title" column="blog_title"/>  
  <association property="author" javaType="Author">  
  <id property="id" column="author_id"/>  
  <result property="username" column="author_username"/>  
  <result property="password" column="author_password"/>  
  <result property="email" column="author_email"/>  
 <result property="bio" column="author_bio"/>  
  <result property="favouriteSection" column="author_favourite_section"/>  
  </association>  
  <collection property="posts" ofType="Post">  
  <id property="id" column="post_id"/>  
  <result property="subject" column="post_subject"/>  
  <association property="author" javaType="Author"/>  
  <collection property="comments" ofType="Comment">  
  <id property="id" column="comment_id"/>  
  </collection>  
  <collection property="tags" ofType="Tag" >  
  <id property="id" column="tag_id"/>  
  </collection>  
  <discriminator javaType="int" column="draft">  
  <case value="1" resultType="DraftPost"/>  
  </discriminator>  
 </collection>  
.  </resultMap>  
```
对于初学者而言，看到这样的一份XML文件，我想内心一定是崩溃的！但是，不要担心，我们日常开发，很少能够遇到这样的场景，并且，相信通过我们一步一步的解释这个配置文档，以后各位也能够运用自如。

在上面的resultMap中存在很多的子元素，下面我们来逐一解释：

“constructor”:类在实例化时，用来注入结果到构造方法中。

“idArg”：ID参数，标记结果作为ID，可以帮助提高整体的效率。

“arg”：注入到构造方法的一个不同结果。

“id”：这个id，类似于数据库的主键，能够帮助提高整体的效率
“result”：即结果字段，其中包括Java对象的属性值，和数据库列名

“association”：复杂类型的结果关联，结果映射能够关联自身，或者关联另一个结果集

“collection”：复杂类型的集合，结果映射自身，或者映射结果集

“discriminator”：使用结果值来决定使用哪个结果映射

“case”：基于某些值的结果映射。嵌入结果映射，这种情形也映射到它本身，因此，能够包含相同的元素，或者参照一个外部的结果映射。

对于resultMap标签，上文的基础用法中我们已经介绍了他的属性含义。但，在此之外，还有一个属性值为：

“autoMapping”：如果出现此配置，Mybatis将会启用或者禁用自动匹配resultMap的功能，这个属性将会在全局范围内覆盖自动匹配机制。默认情况下是没有这个配置的，因此，如果需要，请保持慎重。

下面，我们开始详细说明每一个元素，如果有心急的读者想使用前面增改删查功能，请读者一定按照单元测试的方法推进，千万不要一次性配置大量属性，以免影响学习兴趣。

a.构造方法

```
 <constructor>  
  <idArg column="id" javaType="int"/>  
  <arg column="username" javaType="String"/>  
  </constructor>  
```
尽管对于大部分的DTO对象，以及我们的domain模型，属性值都是能够起到相应的作用，但是，在某些情况下如我们想使用一些固定的类。如：通常情况下的表格中包括一些仅供浏览的数据或者很少改变的数据。Mybatis的构造函数注入功能允许我们在类初始化时就设置某些值，而不暴露其中的public方法。同时Mybatis也支持私有的属性与私有的JavaBeans属性来实现这个目的，尽管这样，一些开发者还是更愿意使用构造函数注入的方式。

例如，程序中我们存在这样一个实体类，如下：

```
1.  public class User {  
2.  //...  
3.  public User(int id, String username) {  
4.  //...  
5.  }  
6.  //...  
7.  }  
```
在Mybatis中，为了向这个构造方法中注入结果，Mybatis需要通过它的参数的类型来表示构造方法。java中，没有反射参数名称的方法，因此，当创建一个构造方法的元素是，必须保证参数是按照顺序排列的，而且，数据类型也必须匹配！

b.关联

``` <association property="author" column="blog_author_id" javaType="Author">  
2.  <id property="id" column="author_id"/>  
3.  <result property="username" column="author_username"/>  
4.  </association>  
```
关联元素用来处理数据模型中的“has-one”关系。如我们上文示例的一个博客有一个用户。关联映射大部分是基于这种应用场景。使用时，我们制定目标属性，可以选用javaType，jdbcType，typeHandler等属性来覆盖结果集合。

关联查询的不同之处是，我们必须告诉告诉Mybatis如何加载关联关系，这里有两种供我们选择的方法：

嵌套查询：即通过执行另一个预期返回复杂类型的SQL语句。

嵌套结果：使用嵌套结果映射来处理联合结果中重复的子集。
在正式的使用之前，我们先来看看这个的属性配置的具体含义，注意，这里的属性配置跟前文基本增改删查中的区别：


 property : 映射到列结果的字段或属性。如果匹配的是存在的,和给定名称相同的 property JavaBeans 的属性, 那么就会使用。 否则 MyBatis 将会寻找给定名称的字段。 这两种情形你可以使用通常点式的复杂属性导航。比如,可以这样映射  :“ username ”, 或 者 映 射 到 一 些 复 杂 的属性 : “address.street.number” 。 
 
javaType : 一个 Java 类的完全限定名,或一个类型别名(参考上面内建类型别名的列 表) 。如果你映射到一个 JavaBean,MyBatis 通常可以断定类型。然而,如 javaType 果你映射到的是 HashMap,那么你应该明确地指定 javaType 来保证所需的行为。 
 jdbcType : 在这个表格之前的所支持的 JDBC 类型列表中的类型。JDBC 类型是仅仅 需要对插入, 更新和删除操作可能为空的列进行处理。这是 JDBC 的需要, jdbcType 而不是 MyBatis 的。如果你直接使用 JDBC 编程,你需要指定这个类型-但 仅仅对可能为空的值。 
 typeHandler : 我们在前面讨论过默认的类型处理器。使用这个属性,你可以覆盖默认的 typeHandler 类型处理器。 这个属性值是类的完全限定名或者是一个类型处理器的实现, 或者是类型别名。 

-------------------------------------------------------------------------------------------------------------------------------------

现在正式的介绍这两种方式：

# <a name="t0"></a>1.嵌套查询


 column : 这是来自数据库的类名,或重命名的列标签的值作为一个输入参数传递给嵌套语句，这和通常传递给 resultSet.getString(columnName)方法的字符串是相同的。
注 意 : 要 处 理 复 合 主 键 , 你 可 以 指 定 多 个 列 名 通 过 column= ” {prop1=col1,prop2=col2} ” 这种语法来传递给嵌套查询语 句。这会引起 prop1 和 prop2 以参数对象形式来设置给目标嵌套查询语句 
select : 另外一个映射语句的 ID,将会按照属性的映射来加载复杂类型。获取的在列属性中指定的列的值将被传递给目标 select 语句作为参数。表格后面 有一个详细的示例。
注 意 : 要 处 理 复 合 主 键 ,  可 以  通 过使用 column= ” {prop1=col1,prop2=col2} ” 这种语法指定多个列名传递给嵌套查询语句。这会导致 prop1 和 prop2 以参数对象形式来设置给目标嵌套查询语句。 
fetchType : 可选的，他的有效值是lazy，eager。如果存在的话，他将在当前映射关系中取代全局变量lazyLoadingEnabled。 

```

  <resultMap id="blogResult" type="Blog">  
 <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>  
  </resultMap>  

  <select id="selectBlog" resultMap="blogResult">  
  SELECT * FROM BLOG WHERE ID = #{id}  
  </select>  

 <select id="selectAuthor" resultType="Author">  
  SELECT * FROM AUTHOR WHERE ID = #{id}  
  </select>  
```

我们有两个查询语句:一个来加载博客,另外一个来加载作者,而且博客的结果映射描 述了“selectAuthor”语句应该被用来加载它的 author 属性。

其他所有的属性将会被自动加载,前提假设它们的列和属性名相匹配。

这种方式很简单, 但是对于大型数据集合和列表将不会表现很好。 问题就是我们熟知的 “N+1 查询问题”。概括地讲,N+1 查询问题可以是这样引起的:

*   执行了一个单独的 SQL 语句来获取结果列表(就是“+1”)。
*   对返回的每条记录,执行了一个查询语句来为每个加载细节(就是“N”)。

这个问题会导致成百上千的 SQL 语句被执行。这通常不是期望的。

MyBatis 能延迟加载这样的查询就是一个好处,因此你可以分散这些语句同时运行的消耗。然而,如果你加载一个列表,之后迅速迭代来访问嵌套的数据,你会调用所有的延迟加载,这样的行为可能是很糟糕的。

所以还有另外一种方法。

# <a name="t1"></a>2.嵌套结果：

首先，我们先来看看有哪些属性能够供我们使用：


 resultMap : 这是结果映射的 ID,可以映射关联的嵌套结果到一个合适的对象图中。这是一种替代方法来调用另外一个查询语句。这允许你联合多个表来合成到 resultMap 一个单独的结果集。这样的结果集可能包含需要被分解的相同的，重复的数据组并且合理映射到一个嵌套的对象图。为了使它变得容易,MyBatis 让你“链接”结果映射,来处理嵌套结果。下面给予一个很容易来仿照例子。 
 columnPrefix : 当连接多个表时，你最好使用列的别名来避免在一个结果集合中出现的名称重复。对于制定的前缀，Mybatis允许我们映射列到外部集合中，具体用法请参照后面的例子 
 notNullColumn : 只有在至少有一个非空列映射到子对象的属性时，才创建一个默认的子对象。通过这个属性，我们可以设置哪一个列必须有值来改变这个行为，此时的Mybatis就会按照这个非空设置来创建一个子对象。多个列存在时，可以通过逗号作为分割符。默认情况下，该属性是不会被设置的，即unset 
 autoMapping : 如果存在此属性的话，Mybatis会在映射到对应属性时启用或者禁用自动映射的功能。这个属性将会在全局范围内覆盖自动映射的功能。
注意：该属性没有对外部结果集造成影响。因此，在select或者结果集合中使用是没有意义的。默认情况下，它是不设置的，即unset

上面的内容是不是让各位已经头晕目眩了？不要急，我们马上就给大家展示一个非常简单的例子来说明上面各个属性是怎么工作的。

```
  <select id="selectBlog" resultMap="blogResult">  
  select  
  B.id            as blog_id,  
  B.title         as blog_title,  
  B.author_id     as blog_author_id,  
  A.id            as author_id,  
  A.username      as author_username,  
  A.password      as author_password,  
  A.email         as author_email,  
  A.bio           as author_bio  
  from Blog B left outer join Author A on B.author_id = A.id  
  where B.id = #{id}  
  </select>  
```
仔细观察这个联合查询，以及采用的保护方法确保了所有结果被唯一的，清晰的命名。这样使得我们的映射非常的简单。

现在，我们来看看如何映射这个结果

```
  <resultMap id="blogResult" type="Blog">  
  <id property="id" column="blog_id" />  
  <result property="title" column="blog_title"/>  
  <association property="author" resultMap="authorResult" />  
  </resultMap>  

  <resultMap id="authorResult" type="Author">  
  <id property="id" column="author_id"/>  
  <result property="username" column="author_username"/>  
  <result property="password" column="author_password"/>  
  <result property="email" column="author_email"/>  
  <result property="bio" column="author_bio"/>  
  </resultMap>  
```
在上面的这个例子中，我们已经观察到博客的“author”关联另外一个“resultMap”结果映射，来加载“Author”实例

特别注意的是：在嵌套结果映射中，元素“id”扮演了一个非常重要的角色。我们应该特别指明一个或者多个属性来唯一的标识这个结果。在实际应用中，如果不指定这个“id”，Mybatis仍然能够继续运行，但是会产生很大的性能消耗。但是，也需要尽可能的少的选择这些属性，数据库的主键显然是一个非常好的选择！

上面的例子使用了外部结果集元素来映射关联。这样的做法，使得id为“Authot”的结果集能够被不断的重用。但是，假如我们没有重用的需求，或者，我们只是想简单的把我们的结果映射到一个单独描述的结果集合当中的话，就不再需要上面的方式书写了，直接嵌套关联结果映射就好。具体的做法如下：

```

  <resultMap id="blogResult" type="Blog">  
  <id property="id" column="blog_id" />  
  <result property="title" column="blog_title"/>  
  <association property="author" javaType="Author">  
  <id property="id" column="author_id"/>  
  <result property="username" column="author_username"/>  
 <result property="password" column="author_password"/>  
  <result property="email" column="author_email"/>  
  <result property="bio" column="author_bio"/>  
 </association>  
  </resultMap>  
```
针对上面的例子，加入这篇博客存在一个“联合作者”又该怎么办呢？具体的做法如下：
```

  <select id="selectBlog" resultMap="blogResult">  
 select  
  B.id            as blog_id,  
  B.title         as blog_title,  
    A.id            as author_id,  
  A.username      as author_username,  
  A.password      as author_password,  
  A.email         as author_email,  
  A.bio           as author_bio,  
  CA.id           as co_author_id,  
  CA.username     as co_author_username,  
  CA.password     as co_author_password,  
  CA.email        as co_author_email,  
  CA.bio          as co_author_bio  
  from Blog B  
  left outer join Author A on B.author_id = A.id  
  left outer join Author CA on B.co_author_id = CA.id  
  where B.id = #{id}  
  </select>  
```
```
  <resultMap id="authorResult" type="Author">  
  <id property="id" column="author_id"/>  
  <result property="username" column="author_username"/>  
  <result property="password" column="author_password"/>  
  <result property="email" column="author_email"/>  
  <result property="bio" column="author_bio"/>  
 </resultMap>  
```
由于结果集当中的列名与查询结果当中列名不一致，我们需要使用明确指定“columnPrefix”来重用这个结果集，以此来映射“联合作者”的查询结果。具体写法如下：
```

  <resultMap id="blogResult" type="Blog">  
  <id property="id" column="blog_id" />  
  <result property="title" column="blog_title"/>  
  <association property="author"  
  resultMap="authorResult" />  
  <association property="coAuthor"  
  resultMap="authorResult"  
  columnPrefix="co_" />  
  </resultMap>  
```


# <a name="t2"></a>多结果集关联


|column :当时用多结果集的这个属性来指定被逗号分隔的列时，将会使得该列与<tt>foreignColumn</tt> 相关联，从而确定了关联列之间的父子关系 
 foreignColumn :  标识出包含foreing keys的列的名称。这个foreing keys的值将会和父类型中指定的列属性的值相匹配
 
 resultSet :  标识这个将会从哪里加载的复杂类型数据的结果集合的名称 |

从3.2.3版本开始，Mybatis提供了另外一种方式来解决“N+1”问题

某些数据库允许在存储过程中返回多个结果集，或者同时执行多个语句并且每一个都返回一个结果集合。这就使得我们可以只用访问数据库一次，且不用使用join，就返回存在关联的数据。

举个例子，执行下面的语句将会返回两个结果集合。第一个返回博客文章的结果集合，第二个返回作者的结果集合

```
  SELECT * FROM BLOG WHERE ID = #{id}  

  SELECT * FROM AUTHOR WHERE ID = #{id}  
```
 我们必须给予每一个结果集合一个指定的名称。方式是：在结果集合中增加一个<tt>resultSets</tt> 属性，来映射语句中被逗号分隔的名称。
```
 <select id="selectBlog" resultSets="blogs,authors" resultMap="blogResult" statementType="CALLABLE">  
 {call getBlogsAndAuthors(#{id,jdbcType=INTEGER,mode=IN})}  
  </select>  
```
现在，我们来指定：数据填充的“author”的集合包含在“authors”的结果集中
```
<resultMap id="blogResult" type="Blog">  
  <id property="id" column="id" />  
  <result property="title" column="title"/>  
  <association property="author" javaType="Author" resultSet="authors" column="author_id" foreignColumn="id">  
  <id property="id" column="id"/>  
  <result property="username" column="username"/>  
  <result property="password" column="password"/>  
  <result property="email" column="email"/>  
  <result property="bio" column="bio"/>  
  </association>  
  </resultMap>  
```
上面所属的这些内容，解决了我们“has-one”的问题。

--------------------------------------------------------------------------------------------------------

### <a name="a"></a>自动映射

正如你在前面一节看到的，在简单的场景下，MyBatis可以替你自动映射查询结果。 如果遇到复杂的场景，你需要构建一个result map。 但是在本节你将看到，你也可以混合使用这两种策略。 让我们到深一点的层面上看看自动映射是怎样工作的。

当自动映射查询结果时，MyBatis会获取sql返回的列名并在java类中查找相同名字的属性（忽略大小写）。 这意味着如果Mybatis发现了_ID_列和_id_属性，Mybatis会将_ID_的值赋给_id_。

通常数据库列使用大写单词命名，单词间用下划线分隔；而java属性一般遵循驼峰命名法。 为了在这两种命名方式之间启用自动映射，需要将 <tt>mapUnderscoreToCamelCase</tt>设置为true。

自动映射甚至在特定的result map下也能工作。在这种情况下，对于每一个result map,所有的ResultSet提供的列， 如果没有被手工映射，则将被自动映射。自动映射处理完毕后手工映射才会被处理。 在接下来的例子中， _id_ 和 _userName_列将被自动映射， _hashed_password_ 列将根据配置映射。

```
<select id="selectUsers" resultMap="userResultMap">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password
  from some_table
  where id = #{id}
</select>
```

```
<resultMap id="userResultMap" type="User">
  <result property="password" column="hashed_password"/>
</resultMap>
```

有三种自动映射等级：

*   <tt>NONE</tt> - 禁用自动映射。仅设置手动映射属性。
*   <tt>PARTIAL</tt> - 将自动映射结果除了那些有内部定义内嵌结果映射的(joins).
*   <tt>FULL</tt> - 自动映射所有。

默认值是<tt>PARTIAL</tt>，这是有原因的。当使用<tt>FULL</tt>时，自动映射会在处理join结果时执行，并且join取得若干相同行的不同实体数据，因此这可能导致非预期的映射。下面的例子将展示这种风险：

```
<select id="selectBlog" resultMap="blogResult">
  select
    B.id,
    B.title,
    A.username,
  from Blog B left outer join Author A on B.author_id = A.id
  where B.id = #{id}
</select>
```

```
<resultMap id="blogResult" type="Blog">
  <association property="author" resultMap="authorResult"/>
</resultMap>

<resultMap id="authorResult" type="Author">
  <result property="username" column="author_username"/>
</resultMap>
```

在结果中_Blog_和_Author_均将自动映射。但是注意_Author_有一个_id_属性，在ResultSet中有一个列名为_id_， 所以Author的id将被填充为Blog的id，这不是你所期待的。所以需要谨慎使用<tt>FULL</tt>。

通过添加<tt>autoMapping</tt>属性可以忽略自动映射等级配置，你可以启用或者禁用自动映射指定的ResultMap。

```
<resultMap id="userResultMap" type="User" autoMapping="false">
  <result property="password" column="hashed_password"/>
</resultMap>
```

<a name="cache"></a>

### <a name="a"></a>缓存

MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制。MyBatis 3 中的缓存实现的很多改进都已经实现了,使得它更加强大而且易于配置。

默认情况下是没有开启缓存的,除了局部的 session 缓存,可以增强变现而且处理循环 依赖也是必须的。要开启二级缓存,你需要在你的 SQL 映射文件中添加一行:

```
<cache/>
```

字面上看就是这样。这个简单语句的效果如下:

*   映射语句文件中的所有 select 语句将会被缓存。
*   映射语句文件中的所有 insert,update 和 delete 语句会刷新缓存。
*   缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
*   根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序 来刷新。
*   缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用。
*   缓存会被视为是 read/write(可读/可写)的缓存,意味着对象检索不是共享的,而 且可以安全地被调用者修改,而不干扰其他调用者或线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改。比如:

```
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

这个更高级的配置创建了一个 FIFO 缓存,并每隔 60 秒刷新,存数结果对象或列表的 512 个引用,而且返回的对象被认为是只读的,因此在不同线程中的调用者之间修改它们会 导致冲突。

可用的收回策略有:

*   <tt>LRU</tt> – 最近最少使用的:移除最长时间不被使用的对象。
*   <tt>FIFO</tt> – 先进先出:按对象进入缓存的顺序来移除它们。
*   <tt>SOFT</tt> – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
*   <tt>WEAK</tt> – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

默认的是 LRU。

flushInterval(刷新间隔)可以被设置为任意的正整数,而且它们代表一个合理的毫秒 形式的时间段。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。

size(引用数目)可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的 可用内存资源数目。默认值是 1024。

readOnly(只读)属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓 存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存 会返回缓存对象的拷贝(通过序列化) 。这会慢一些,但是安全,因此默认是 false。

#### <a name="a"></a>使用自定义缓存

除了这些自定义缓存的方式, 你也可以通过实现你自己的缓存或为其他第三方缓存方案 创建适配器来完全覆盖缓存行为。

```
<cache type="com.domain.something.MyCustomCache"/>
```

这个示 例展 示了 如何 使用 一个 自定义 的缓 存实 现。type 属 性指 定的 类必 须实现 org.mybatis.cache.Cache 接口。这个接口是 MyBatis 框架中很多复杂的接口之一,但是简单 给定它做什么就行。

```
public interface Cache {
  String getId();
  int getSize();
  void putObject(Object key, Object value);
  Object getObject(Object key);
  boolean hasKey(Object key);
  Object removeObject(Object key);
  void clear();
}
```

要配置你的缓存, 简单和公有的 JavaBeans 属性来配置你的缓存实现, 而且是通过 cache 元素来传递属性, 比如, 下面代码会在你的缓存实现中调用一个称为 “setCacheFile(String file)” 的方法:

```
<cache type="com.domain.something.MyCustomCache">
  <property name="cacheFile" value="/tmp/my-custom-cache.tmp"/>
</cache>
```

你可以使用所有简单类型作为 JavaBeans 的属性,MyBatis 会进行转换。 And you can specify a placeholder(e.g. <tt>${cache.file}</tt>) to replace value defined at [configuration properties](configuration.html#properties).

从3.4.2版本开始，MyBatis已经支持在所有属性设置完毕以后可以调用一个初始化方法。如果你想要使用这个特性，请在你的自定义缓存类里实现 <tt>org.apache.ibatis.builder.InitializingObject</tt> 接口。

```
public interface InitializingObject {
  void initialize() throws Exception;
}
```

记得缓存配置和缓存实例是绑定在 SQL 映射文件的命名空间是很重要的。因此,所有 在相同命名空间的语句正如绑定的缓存一样。 语句可以修改和缓存交互的方式, 或在语句的 语句的基础上使用两种简单的属性来完全排除它们。默认情况下,语句可以这样来配置:

```
<select ... flushCache="false" useCache="true"/>
<insert ... flushCache="true"/>
<update ... flushCache="true"/>
<delete ... flushCache="true"/>
```

因为那些是默认的,你明显不能明确地以这种方式来配置一条语句。相反,如果你想改 变默认的行为,只能设置 flushCache 和 useCache 属性。比如,在一些情况下你也许想排除 从缓存中查询特定语句结果,或者你也许想要一个查询语句来刷新缓存。相似地,你也许有 一些更新语句依靠执行而不需要刷新缓存。

#### <a name="a"></a>参照缓存

回想一下上一节内容, 这个特殊命名空间的唯一缓存会被使用或者刷新相同命名空间内 的语句。也许将来的某个时候,你会想在命名空间中共享相同的缓存配置和实例。在这样的 情况下你可以使用 cache-ref 元素来引用另外一个缓存。

```
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```

## 结束

谢谢各位。