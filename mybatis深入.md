# resultMap

> 解决的是多表查询和对象关系映射的问题
> 在查询时，因为多张表的结果放到了一起
> 所以多张表中的id就可能会重名
> 那么哪一个id是哪一个对象的哪个属性，需要考虑
> 如何处理？使用SQL语句时，让把个id都变得不一样

```mysql
-- 此处ctid和pid就是为了解决同名id问题
SELECT 
*,
tb_cart.id as ctid,
tb_product.id as pid
FROM tb_cart INNER JOIN tb_product ON tb_cart.pid=tb_product.id;
```

```xml
    <!-- id子标签，描述了结果集中的主键和对象中的属性 -->
	<!-- column描述的是结果集中的字段名 -->
	<!-- property对应的是对象中的属性 -->
    <resultMap id="cartResultMap" type="Cart">
        <id column="ctid" property="id"/>
    </resultMap>
```

## resultMap必须要结合使用

> 可以单独使用，但单独使用没有意义
> 所以一个resultMap用于描述Cart
> 另一个resultMap用于描述Product

```xml
    <resultMap id="cartResultMap" type="Cart">
        <id column="ctid" property="id"/>
        <result column="amount" property="amount"/>
        <result column="count" property="count"/>
        <result column="uid" property="uid"/>
        <result column="pid" property="pid"/>
        <association property="product" resultMap="productResultMap"/>
    </resultMap>

    <resultMap id="productResultMap" type="Product">
        <id column="pid" property="id"/>
        <result column="name" property="name"/>
        <result column="pubdate" property="pubdate"/>
        <result column="picture" property="picture"/>
        <result column="price" property="price"/>
        <result column="star" property="star"/>
        <result column="summary" property="summary"/>
        <result column="cid" property="cid"/>
    </resultMap>
```

## resultMap做减法

> column和property一样的话，尽量自动
> 用resultMap标签的autoMapping属性
> 但是每个resultMap都得写，所以希望开启全局设置，所有resultMap默认打开autoMapping

```xml
    <resultMap id="cartResultMap" type="Cart" autoMapping="true">
        <id column="ctid" property="id"/>
        <association property="product" resultMap="productResultMap"/>
    </resultMap>

    <resultMap id="productResultMap" type="Product" autoMapping="true">
        <id column="pid" property="id"/>
    </resultMap>
```

## resultMap自动映射全局配置

- `mybatis-config.xml`

```xml
    <settings>
        <setting name="autoMappingBehavior" value="FULL"/>
    </settings>
```

## resultMap对多关联

```xml
    <resultMap id="userResultMap" type="User">
        <id column="uid" property="id"/>
        <collection property="carts" resultMap="com.futureweaver.mapper.CartMapper.cartResultMap"/>
    </resultMap>
```

# 懒加载

> user对象中的carts属性，在使用时(getCarts)，才去查询
> 比较懒，催了才去查，所以叫懒加载，也叫延迟加载

```xml
    <resultMap id="userResultMapLazy" type="User">
        <id column="uid" property="id"/>

		<!-- user里面的carts属性，在调用时，需要去数据库里查，使用CartMapper的queryByUid方法去查 -->
		<!-- 这个方法是需要参数的，把结果集中的uid，传过去 -->
        <collection property="carts" select="com.futureweaver.mapper.CartMapper.queryByUid" column="uid"/>
    </resultMap>
```

# 动态SQL

- if
  > 条件判断，满足的话，才加SQL子句

- where
  > 自动判断是否需要添加WHERE子句，并且会自动判断是否删除AND或OR
  > 前提AND或OR，要放到SQL子句的前面，不能放到后面

- set
  > 自动判断是否需要添加SET子句，并且会自动判断是否删除逗号

- foreach
  > 自动遍历集合，可以指定分隔符、开始字符、结束字符、循环体中的变量名...

- `sql/include`
  > sql用于定义SQL片断
  > include用于引用SQL片断
  > 一般就是SQL片断的复用(重复使用)

