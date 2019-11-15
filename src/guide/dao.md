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

### 实体类代码生成工具

实体类也可以通过框架提供的内置工具进行生成，当前版本使用的是*FreeMarker*模板引擎进行生成，所以需要确认在使用生成工具类时正确引入了*FreeMarker*依赖，依赖如下：

```xml
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
</dependency>
```

调用生成代码如下：

```java
Map<String, Object> _dataModel = new HashMap<>();
// 实体类所属包名
_dataModel.put("entityPackage", "com.wee0.box.examples.multiModule.module1.entity");
// 实体类继承的公共基类
_dataModel.put("entityBase", "com.wee0.box.examples.multiModule.module1.entity.BaseEntity");
// 作者名称
_dataModel.put("author", "baihw");
// 文件创建时间
_dataModel.put("createDate", DateUtils.getCurrentDate());
// 生成的实体类保存文件夹位置
File _entityDir = new File("D:/test");
// 不需要在实体类中包含的列，通常是因为在公共基类中已经统一定义了。
Set<String> _excludeColumns = new HashSet<>(6);
_excludeColumns.add("ID");
_excludeColumns.add("CREATE_TIME");
_excludeColumns.add("CREATE_USER");
_excludeColumns.add("UPDATE_TIME");
_excludeColumns.add("UPDATE_USER");
_excludeColumns.add("IS_DELETED");
// 不需要在实体类中包含的表，这里排除关系表的生成。
Set<String> _excludeTables = new HashSet<>(2);
_excludeTables.add("sys_user_role_rel");
_excludeTables.add("sys_role_permission_rel");
// 自定义命名策略
ISqlTemplateHelper.INamePolicy _namePolicy = new ISqlTemplateHelper.INamePolicy() {
    @Override
    public String renameEntity(String original, String current) {
        // 统一加上Entity后缀。
        return current + "Entity";
   }
};
// 调用数据库模板助手类实例生成实体类的方法
SqlTemplateHelper.impl().generateEntities(_dataModel, "entity.ftl", _entityDir, _excludeColumns, _excludeTables, _namePolicy);
```

*entity.ftl* 模板文件可以从示例项目*web工程*的 *src/test/resources/templates/entity.ftl* 位置获取，也可以通过调整模板文件来定制自己的最终生成格式，示例项目*web工程*的 *TestGenerate.java* 源码中有此部分的测试代码。

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

## 分页支持

**使用方法**

默认对名称以“Page”结尾的查询方法进行拦截统一处理，可以通过配置修改为自定义的规则，如下：
```xml
<plugin interceptor="com.wee0.box.sql.dao.mybatis.MybatisPageInterceptor">
    <property name="pageMethodSuffix" value="ByPage"/>
</plugin>
```

Java代码使用示例：
```java
Map<String, Object> _params = PageHelper.impl().createPageParams(2, 10);
// 增加其它参数
_params.put("isDeleted", false);
// 结果集返回的是当前页的数据
List<SysUserEntity> _data = sysUserDao.queryAllByPage(_params);
// 需要分页相关详细数据，如页数，总行数等可以从IPage对象获取。
IPage _page = PageHelper.impl().parseMap(_params);
System.out.println("_page: " + _page);
```

## 事务支持

**使用方法**

```java
boolean _result = TxManger.impl().tx(() -> {
    // 修改其它属性，不修改唯一标识 
    _sysUserDao.updateEntity(_user);
    // 唯一标识重复，插入失败
    _sysUserDao.insertEntity(_user);
});
// 如果dao方法执行发生异常，将会回滚事务，打印日志，然后返回false。
System.out.println("transaction result: " + _result);
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

