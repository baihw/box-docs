# 数据访问层

数据访问层当前版本仅提供了基于mybatis的实现，框架提供了基本的单表增、删、改、查功能的自动适配，只需要继承IBaseDao即可使用，不需要写mapper.xml文件。如果某个自动适配的结果与预期不一致时，可以选择显式的在mapper.xml文件中进行定义覆盖。当mapper.xml中存在对应方法的语句定义时，mapper.xml文件中显式声明的优先级更高。关于mybatis的Mapper xml编写方法，这里就不再赘述了，参考 [官方文档](https://mybatis.org/mybatis-3/sqlmap-xml.html) 或者 [官方中文翻译文档](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html)。

## 实体类

```java
@BoxTable(name = "SYS_USER")
public class SysUserEntity extends BaseEntity {

    /**
     * 用户名
     */
    @BoxColumn(name = "USER_NAME")
    protected String userName;

    /**
     * 密码
     */
    protected String password;

    /**
     * 用户昵称
     */
    @BoxColumn(name = "NICK_NAME")
    protected String nickName;

    /**
     * 性别
     */
    protected int sex;

    /**
     * 邮箱
     */
    protected String email;

    /**
     * 手机
     */
    protected String mobile;
    
    // 此处省略了 geeter / setter 方法
}
```

- BaseEntity - 一个基础实体对象，封装了基本上每个表都会具备的一些通用属性。
- BoxTable - 表标记注解，当实际的表名与实体类类名不一致时，可通过name属性指定映射关系。
- BoxColumn - 列标记注解，当实际的列名与实体类属性名不一致时，可通过name属性指定映射关系。

### BaseEntity主要代码如下

```java
@BoxIgnore
public class BaseEntity extends AbstractEntity<String> {

    @BoxId
    @BoxColumn(name = "ID")
    protected String id;

    /**
     * 创建时间
     */
    @BoxColumn(name = "CREATE_TIME")
    protected Date createTime;

    /**
     * 创建用户
     */
    @BoxColumn(name = "CREATE_USER")
    protected String createUser;

    /**
     * 修改时间
     */
    @BoxColumn(name = "UPDATE_TIME")
    protected Date updateTime;

    /**
     * 修改用户
     */
    @BoxColumn(name = "UPDATE_USER")
    protected String updateUser;

    /**
     * 是否标记删除:0-未标记删除；1-标记删除；
     */
    @BoxColumn(name = "IS_DELETED")
    protected Boolean isDeleted;
    
    // 此处省略了 geeter / setter 方法
}
```

## Dao接口

```java
@BoxDao
public interface SysUserDao extends IBaseDao<SysUserEntity, String> {
    int updatePassword(SysUserEntity sysUserEntity);
}
```

- BoxDao - Dao对象注解，标注了此注解的接口在启动时进行扫描处理。
- IBaseDao - 一个包含了简单增、删、改、查操作的基础接口，继承自此接口中的方法，不需要自己编写Mapper xml实现，由框架根据数据库环境自动适配，当前版本支持的数据库有Mysql、Oracle。
- String - 主键数据类型

此例子中，只需要在Mapper xml中提供自已定义的updatePassword方法的sql语句即可。

### IBaseDao接口定义代码如下

```java
/**
     * 插入数据
     *
     * @param entity 实体对象
     * @return 受影响的记录数
     */
    <S extends T> int insertEntity(S entity);

    /**
     * 修改数据
     *
     * @param entity 实体对象
     * @return 受影响的记录数
     */
    <S extends T> int updateEntity(S entity);

    /**
     * 删除指定标识数据
     *
     * @param id 唯一标识
     * @return 受影响的记录数
     */
    int deleteById(ID id);

    /**
     * 删除指定标识数据
     *
     * @param ids 唯一标识集合
     * @return 受影响的记录数
     */
    int deleteByIds(List<? extends ID> ids);

    /**
     * 删除所有数据
     *
     * @return 受影响的记录数
     */
    int deleteAll();

    /**
     * 统计总数量
     *
     * @return 总数量
     */
    long countAll();

    /**
     * 是否存在指定标识数据
     *
     * @param id 唯一标识
     * @return true / false
     */
    Boolean existsById(ID id);

    /**
     * 查询指定标识数据
     *
     * @param id 唯一标识
     * @return 数据
     */
    T queryById(ID id);

    /**
     * 查询指定标识数据
     *
     * @param ids 唯一标识集合
     * @return 数据
     */
    List<T> queryByIds(List<? extends ID> ids);

    /**
     * 查询所有数据
     *
     * @return 所有数据
     */
    List<T> queryAll();

    /**
     * 查询分页数据
     *
     * @param pageNum  页码
     * @param pageSize 每页大小
     * @return 分页数据
     */
    List<T> queryAllByPage(int pageNum, int pageSize);
```

## mybatis单文件多数据库语法支持

**适用场景**

一些被其它项目依赖的项目，如果对不同环境进行区别打包发布的话，一来比较麻烦，二来也会导致其它项目在引用时需要根据不同的环境去切换依赖，如oracle环境依赖 *proj-x.y.z-oracle* 版本，mysql环境依赖 *proj-x.y.z-mysql* 版本。由于项目中大部分使用的都是标准sql语法，需要针对数据库环境进行定制的语法点并不多，所以我们可以选择此方案。反之，如果差异性太大，没有多少可重用语法的情况下，建议使用不同数据库不同目录的处理方式，但这并不是个单选问题，它们是可以并存的，所以不用担心必须做出唯一选择，请在合适的场景选择合适的方案。

**使用方法**

在需要对特定数据库进行特殊处理的语句节点加入数据库标识属性*databaseId*，当前可选的值有：mysql, postgres, oracle, db2, sybase, microsoft, h2。

参考以下*mapper.xml*文件示例

```xml
<mapper namespace="com.wee0.box.examples.dao.SysUserDao">
  <resultMap id="userMap" type="java.util.Map">
    <result column="ID" property="id" javaType="java.lang.Integer"/>
  </resultMap>
  <sql id="column_list">
    ID, TITLE
  </sql>
  <!-- oracle数据库此节点生效 -->
  <sql id="where_filters" databaseId="oracle">
    <where>
      <if test="startDt != null and startDt!=''">
      <![CDATA[ AND CREATE_DATE >= to_date(#{startDt},'yyyy-mm-dd') ]]>
      </if>
    </where>
  </sql>
  <!-- 其它数据库此节点生效 -->
  <sql id="where_filters">
    <where>
      <if test="startDt != null and startDt!=''">
        <![CDATA[ AND CREATE_DATE >= #{startDt} ]]>
      </if>
    </where>
  </sql>
  <select id="findUserList" parameterType="map" resultMap="userMap">
    SELECT
    <include refid="column_list" />
    FROM SYS_USER
    <include refid="where_filters" />
    ORDER BY CREATE_DATE DESC
  </select>

  <!-- 试着执行test方法看看效果吧 -->
  <select id="test" databaseId="oracle">select 1 from dual</select>
  <select id="test" databaseId="mysql">select 2</select>
  <select id="test">select 1</select>
</mapper>    
```

## mybatis分目录多数据库支持

待补充。。。

