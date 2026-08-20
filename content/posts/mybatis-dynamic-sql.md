---
title: 重新定义MyBatis动态SQL
description: 使用 MyBatis Dynamic SQL 重构动态查询的实践。
publishDate: 2019-07-22 16:34:50
tags:
  - MyBatis
---

转载自 [Mybatis Dynamic SQL - 重新定义 Mybatis 动态 SQL](https://blog.olowolo.com/post/new-mybatis-dynamic-sql/)

**通过全新的 ByExample 方法快速了解一下这是什么：**

```java
// where (id < ? and employed = ?) or occupation like ? order by id DESC
List<Employee> employees = mapper.selectByExample()
    .where(id, isLessThan(10), and(employed, isEqualTo("foo")))
    .or(occupation, isLike("b%"))
    .orderBy(id.descending())
    .build()
    .execute();
```

# 快速开始

- 新的动态SQL需要额外依赖 `mybatis-dynamic-sql`
- Java 8 及以上
- MyBatis 3.4.2 及以上

```xml
<dependency>
  <groupId>org.mybatis.dynamic-sql</groupId>
  <artifactId>mybatis-dynamic-sql</artifactId>
  <version>1.1.0</version>
</dependency>
```

# 使用 MyBatis Generator

:::note[Info]
MyBatis Generator 1.3.6 及以上
:::

:::note[Info]
1.3.6 在动态 SQL 模式下存在数个已修复的 BUG，建议使用 1.3.7 及以上版本。
:::

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-maven-plugin</artifactId>
      <version>1.3.7</version>
    </plugin>
  </plugins>
</build>
```

只需要将 context 的 targetRuntime 属性更改为 MyBatis3DynamicSQL 即可生成新的动态 SQL

```xml
<generatorConfiguration>
  ...
  <context ... targetRuntime="MyBatis3DynamicSQL" ...>
    ...
  </context>
</generatorConfiguration>
```

MyBatis3DynamicSQL 模式具有以下特性：

- 不再生成 XML，`<sqlMapGenerator>` 将被忽略
- mapper 总是以注解形式实现，`<javaClientGenerator>` 的 type 属性将被忽略
- model 类总是以 FLAT 形式生成

## 实例演示

我们用下面这张表来演示一下实际的生成结果。

```sql
create table employee (
  id         INT         NOT NULL,
  first_name VARCHAR(30) NOT NULL,
  last_name  VARCHAR(30) NOT NULL,
  birth_date DATE        NOT NULL,
  employed   VARCHAR(3)  NOT NULL,
  occupation VARCHAR(30) NULL,
  PRIMARY KEY (id)
);
```

MyBatis Generator 将会生成下述三个文件：

- 与表对应的 model 类 [`Employee.java`](/code/new-mybatis-dynamic-sql/Employee.java)
- 定义了表信息和列信息的 support 类 [`EmployeeDynamicSqlSupport.java`](/code/new-mybatis-dynamic-sql/EmployeeDynamicSqlSupport.java)
- 以注解形式实现的 mapper 接口 [`EmployeeMapper.java`](/code/new-mybatis-dynamic-sql/EmployeeMapper.java)

:::note[Info]
为了节省版面移除了一些代码，源代码可点击查看
:::

我们暂时先不去关心具体的内部实现，大致看一下生成文件的样貌，之后先来了解如何使用这些文件。

### model

```java
public class Employee {
  private Integer id;
  private String firstName;
  private String lastName;
  private LocalDate birthDate;
  private String employed;
  private String occupation;

  // getter and setter ...
}
```

### support

```java
public final class EmployeeDynamicSqlSupport {
  public static final Employee employee = new Employee();
  public static final SqlColumn<Integer> id = employee.id;
  public static final SqlColumn<String> firstName = employee.firstName;
  public static final SqlColumn<String> lastName = employee.lastName;
  public static final SqlColumn<LocalDate> birthDate = employee.birthDate;
  public static final SqlColumn<String> employed = employee.employed;
  public static final SqlColumn<String> occupation = employee.occupation;

  public static final class Employee extends SqlTable {
    public final SqlColumn<Integer> id = column("id", JDBCType.INTEGER);
    public final SqlColumn<String> firstName = column("first_name", JDBCType.VARCHAR);
    public final SqlColumn<String> lastName = column("last_name", JDBCType.VARCHAR);
    public final SqlColumn<LocalDate> birthDate = column("birth_date", JDBCType.DATE);
    public final SqlColumn<String> employed = column("employed", JDBCType.VARCHAR);
    public final SqlColumn<String> occupation = column("occupation", JDBCType.VARCHAR);
    public Employee() {
       super("employee");
    }
  }
}
```

### mapper

```java
@Mapper
public interface EmployeeMapper {

  @SelectProvider(type=SqlProviderAdapter.class, method="select")
  long count(SelectStatementProvider selectStatement);

  @DeleteProvider(type=SqlProviderAdapter.class, method="delete")
  int delete(DeleteStatementProvider deleteStatement);

  @InsertProvider(type=SqlProviderAdapter.class, method="insert")
  int insert(InsertStatementProvider<Employee> insertStatement);

  @SelectProvider(type=SqlProviderAdapter.class, method="select")
  @ResultMap("EmployeeResult")
  Employee selectOne(SelectStatementProvider selectStatement);

  @SelectProvider(type=SqlProviderAdapter.class, method="select")
  @Results(id="EmployeeResult", value = {
      @Result(column="id", property="id", jdbcType=JdbcType.INTEGER, id=true),
      @Result(column="first_name", property="firstName", jdbcType=JdbcType.VARCHAR),
      @Result(column="last_name", property="lastName", jdbcType=JdbcType.VARCHAR),
      @Result(column="birth_date", property="birthDate", jdbcType=JdbcType.DATE),
      @Result(column="employed", property="employed", jdbcType=JdbcType.VARCHAR),
      @Result(column="occupation", property="occupation", jdbcType=JdbcType.VARCHAR)
  })
  List<Employee> selectMany(SelectStatementProvider selectStatement);

  @UpdateProvider(type=SqlProviderAdapter.class, method="update")
  int update(UpdateStatementProvider updateStatement);

  default QueryExpressionDSL<MyBatis3SelectModelAdapter<Long>> countByExample() {
    return SelectDSL.selectWithMapper(this::count, SqlBuilder.count())
        .from(employee);
  }

  default DeleteDSL<MyBatis3DeleteModelAdapter<Integer>> deleteByExample() {
    return DeleteDSL.deleteFromWithMapper(this::delete, employee);
  }

  default int deleteByPrimaryKey(Integer id_) {
    return DeleteDSL.deleteFromWithMapper(this::delete, employee)
        .where(id, isEqualTo(id_))
        .build()
        .execute();
  }

  default int insert(Employee record) {
    return insert(SqlBuilder.insert(record)
        .into(employee)
        .map(id).toProperty("id")
        .map(firstName).toProperty("firstName")
        .map(lastName).toProperty("lastName")
        .map(birthDate).toProperty("birthDate")
        .map(employed).toProperty("employed")
        .map(occupation).toProperty("occupation")
        .build()
        .render(RenderingStrategy.MYBATIS3));
  }

  default int insertSelective(Employee record) {
    return insert(SqlBuilder.insert(record)
        .into(employee)
        .map(id).toPropertyWhenPresent("id", record::getId)
        .map(firstName).toPropertyWhenPresent("firstName", record::getFirstName)
        .map(lastName).toPropertyWhenPresent("lastName", record::getLastName)
        .map(birthDate).toPropertyWhenPresent("birthDate", record::getBirthDate)
        .map(employed).toPropertyWhenPresent("employed", record::getEmployed)
        .map(occupation).toPropertyWhenPresent("occupation", record::getOccupation)
        .build()
        .render(RenderingStrategy.MYBATIS3));
  }

  default QueryExpressionDSL<MyBatis3SelectModelAdapter<List<Employee>>> selectByExample() {
    return SelectDSL.selectWithMapper(this::selectMany, id, firstName, lastName, birthDate, employed, occupation)
        .from(employee);
  }

  default QueryExpressionDSL<MyBatis3SelectModelAdapter<List<Employee>>> selectDistinctByExample() {
    return SelectDSL.selectDistinctWithMapper(this::selectMany, id, firstName, lastName, birthDate, employed, occupation)
        .from(employee);
  }

  default Employee selectByPrimaryKey(Integer id_) {
    return SelectDSL.selectWithMapper(this::selectOne, id, firstName, lastName, birthDate, employed, occupation)
        .from(employee)
        .where(id, isEqualTo(id_))
        .build()
        .execute();
  }

  default UpdateDSL<MyBatis3UpdateModelAdapter<Integer>> updateByExample(Employee record) {
    return UpdateDSL.updateWithMapper(this::update, employee)
        .set(id).equalTo(record::getId)
        .set(firstName).equalTo(record::getFirstName)
        .set(lastName).equalTo(record::getLastName)
        .set(birthDate).equalTo(record::getBirthDate)
        .set(employed).equalTo(record::getEmployed)
        .set(occupation).equalTo(record::getOccupation);
  }

  default UpdateDSL<MyBatis3UpdateModelAdapter<Integer>> updateByExampleSelective(Employee record) {
    return UpdateDSL.updateWithMapper(this::update, employee)
        .set(id).equalToWhenPresent(record::getId)
        .set(firstName).equalToWhenPresent(record::getFirstName)
        .set(lastName).equalToWhenPresent(record::getLastName)
        .set(birthDate).equalToWhenPresent(record::getBirthDate)
        .set(employed).equalToWhenPresent(record::getEmployed)
        .set(occupation).equalToWhenPresent(record::getOccupation);
  }

  default int updateByPrimaryKey(Employee record) {
    return UpdateDSL.updateWithMapper(this::update, employee)
        .set(firstName).equalTo(record::getFirstName)
        .set(lastName).equalTo(record::getLastName)
        .set(birthDate).equalTo(record::getBirthDate)
        .set(employed).equalTo(record::getEmployed)
        .set(occupation).equalTo(record::getOccupation)
        .where(id, isEqualTo(record::getId))
        .build()
        .execute();
  }

  default int updateByPrimaryKeySelective(Employee record) {
    return UpdateDSL.updateWithMapper(this::update, employee)
        .set(firstName).equalToWhenPresent(record::getFirstName)
        .set(lastName).equalToWhenPresent(record::getLastName)
        .set(birthDate).equalToWhenPresent(record::getBirthDate)
        .set(employed).equalToWhenPresent(record::getEmployed)
        .set(occupation).equalToWhenPresent(record::getOccupation)
        .where(id, isEqualTo(record::getId))
        .build()
        .execute();
  }
}
```

## 如何使用

### 未变的方法

以下方法跟之前生成的代码具有一样的效果：

- `deleteByPrimaryKey`
- `insert` - null 属性将会被插入
- `insertSelective` - 忽略 null
- `selectByPrimaryKey`
- `updateByPrimaryKey` - null 属性将会被设置
- `updateByPrimaryKeySelective` - 忽略 null

### 全新的 ByExample

而 ByExample 方法则完全是另一种工作方式。像是下面这样：

```java
// select count(*) from employee
long employeeRows = mapper.countByExample()
    .build()
    .execute();
```

未指定 WHERE 子句时自然是对表中所有行生效。

为了定制 WHRER 子句，在需要使用 mapper 接口的类中写入以下两条导入语句：

```java
import static com.your.project.mapper.EmployeeDynamicSqlSupport.*;
import static org.mybatis.dynamic.sql.SqlBuilder.*;
```

- 前者导入生成的 support 类，这样我们就能够使用 `id`, `employee.id` 这样的字段
- 后者提供了用于构造 SQL 语句的 `or`, `and`, `isEqualTo` 等方法

```java
import static com.your.project.mapper.EmployeeDynamicSqlSupport.*;
import static org.mybatis.dynamic.sql.SqlBuilder.*;

// 举几个栗子
public class SomeService {

  // 一个简单的例子
  public void simpleWhere {
    ...
    // select id, first_name, last_name, birth_date, employed, occupation
    // from employee where (first_name = ? or first_name = ?)
    List<Employee> employees = mapper.selectByExample()
        .where(firstName, isEqualTo("Bob"), or(firstName, isEqualTo("Alice")))
        .build()
        .execute();
    ...
  }

  // 一个稍微复杂些的例子
  public void complexWhere {
    ...
    // select id, first_name, last_name, birth_date, employed, occupation
    // from employee where (first_name = ? or first_name = ?) and birth_date >= ? order by birth_date DESC
    List<Employee> employees = mapper.selectByExample()
        .where(employee.firstName, isEqualTo("Bob"), or(employee.firstName, isEqualTo("Alice")))
        .and(employee.birthDate, isGreaterThanOrEqualTo(LocalDate.of(1990, 1, 1)))
        .orderBy(employee.birthDate.descending())
        .build()
        .execute();
    ...
  }
}
```

# MyBatis Dynamic SQL

:::tip[Tip]
这一部分的内容主要来自 [MyBatis Dynamic SQL 文档](http://www.mybatis.org/mybatis-dynamic-sql/docs/introduction.html)，只少不多，直接阅读文档是个好主意的说
:::

正如 MyBatis Generator 生成的文件，为了使用 MyBatis Dynamic SQL 你至少需要这三个文件：

- model 类
- 定义了表信息和列信息的 support 类
- 基于 XML 或注解实现的 mapper 接口

model 类和 support 类的编写可以参考上述生成的文件，而基本的 mapper 接口实际上只需要下述这些代码。

```java
@Mapper
public interface EmployeeMapper {

  @SelectProvider(type=SqlProviderAdapter.class, method="select")
  long count(SelectStatementProvider selectStatement);

  @DeleteProvider(type=SqlProviderAdapter.class, method="delete")
  int delete(DeleteStatementProvider deleteStatement);

  @InsertProvider(type=SqlProviderAdapter.class, method="insert")
  int insert(InsertStatementProvider<Employee> insertStatement);

  @SelectProvider(type=SqlProviderAdapter.class, method="select")
  @ResultMap("EmployeeResult")
  Employee selectOne(SelectStatementProvider selectStatement);

  @SelectProvider(type=SqlProviderAdapter.class, method="select")
  @Results(id="EmployeeResult", value = {
      @Result(column="id", property="id", jdbcType=JdbcType.INTEGER, id=true),
      @Result(column="first_name", property="firstName", jdbcType=JdbcType.VARCHAR),
      @Result(column="last_name", property="lastName", jdbcType=JdbcType.VARCHAR),
      @Result(column="birth_date", property="birthDate", jdbcType=JdbcType.DATE),
      @Result(column="employed", property="employed", jdbcType=JdbcType.VARCHAR),
      @Result(column="occupation", property="occupation", jdbcType=JdbcType.VARCHAR)
  })
  List<Employee> selectMany(SelectStatementProvider selectStatement);

  @UpdateProvider(type=SqlProviderAdapter.class, method="update")
  int update(UpdateStatementProvider updateStatement);
}
```

回顾一下生成的 mapper 接口，你会发现其他 default 方法实际上都是对这些方法的封装。为了使用这些原始方法，需要传入一个完整的语句，像是下面这样：

```java
SelectStatementProvider selectStatement = select(id, firstName, lastName, birthDate, employed, occupation)
    .from(employee)
    .where(firstName, isEqualTo("Bob"), or(firstName, isEqualTo("Alice")))
    .build()
    .render(RenderingStrategy.MYBATIS3);

List<Employee> employees = mapper.selectMany(selectStatement);
```

## WHERE

- WHRER 子句支持各种各样的条件（参考[附录 I](#appendix-01)）
- 能够任意组合 and 与 or
- 大多数条件也支持子查询

```java
// select id, first_name, last_name, birth_date, employed, occupation
// from employee where id in (select id from employee where (first_name = ? or first_name = ?))
SelectStatementProvider selectStatement = select(id, firstName, lastName, birthDate, employed, occupation)
    .from(employee)
    .where(id, isIn(select(id).from(employee).where(firstName,isEqualTo("Bob"), or(firstName, isEqualTo("Alice")))))
    .build()
    .render(RenderingStrategy.MYBATIS3)
```

## SELECT

目前已经支持的：

1. 一些典型的 SELECT 元素 (SELECT, DISTINCT, FROM, JOIN, WHERE, GROUP BY, UNION, ORDER BY)
2. 每条 SELECT 语句都能为表设置别名
3. 每条 SELECT 语句都能为列设置别名
4. 一些聚集函数 (avg, min, max, sum)
5. 表连接 (INNER, LEFT OUTER, RIGHT OUTER, FULL OUTER)
6. WHERE 子句中的子查询，如 `where foo in (select foo from foos where id < 36)`

尚未支持的:

1.  WITH 表达式
2.  HAVING 表达式
3.  SELECT 另一 SELECT 的结果，如 `select count(*) from (select foo from foos where id < 36)`
4.  INTERSECT，EXCEPT 等。

### JOIN 查询

```java
// select employee.id, employee.first_name, employee.last_name, employee.birth_date, employee.employed, employee.occupation, employee_age.age
// from employee join employee_age on employee.id = employee_age.id
SelectStatementProvider selectStatement = select(employee.id, firstName, lastName, birthDate, employed, occupation, age)
    .from(employee)
    .join(employeeAge).on(employee.id, equalTo(employeeAge.id))
    .build()
    .render(RenderingStrategy.MYBATIS3)
```

如果有必要，你可以为表指定别名。如果没有指定别名，将会使用 `tableName.column` 这种形式。

可以在单个语句中连接多个表。例如:

```java
// select e.id, e.first_name, e.last_name, e.birth_date, e.employed, e.occupation, ea.age, es.salary
// from employee e join employee_age ea on e.id = ea.id join employee_salary es on e.id = es.id where e.id = ?
SelectStatementProvider selectStatement = select(employee.id, firstName, lastName, birthDate, employed, occupation, age, salary)
    .from(employee, "e")
    .join(employeeAge, "ea").on(employee.id, equalTo(employeeAge.id))
    .join(employeeSalary, "es").on(employee.id, equalTo(employeeSalary.id))
    .where(id, isEqualTo(2))
    .build()
    .render(RenderingStrategy.MYBATIS3)
```

由于 MyBatis 注解的局限性，联表查询时你可能需要在 XML 中定义 resultMap。这是唯一需要 XML 的情况。

:::note[Note]
文档的建议是：resultMap 应该是 XML 文件中唯一出现的元素。（但也只是文档的建议→_→）
:::

### UNION 查询

```java
// select id, first_name, last_name, birth_date, employed, occupation from employee
// union
// select distinct id, first_name, last_name, birth_date, employed, occupation from employee
// order by id
SelectStatementProvider selectStatement = select(id, firstName, lastName, birthDate, employed, occupation)
    .from(employee)
    .union()
    .selectDistinct(id, firstName, lastName, birthDate, employed, occupation)
    .from(employee)
    .orderBy(id)
    .build()
    .render(RenderingStrategy.MYBATIS3)
```

支持任意数量的 UNION 查询，但只允许一个 ORDER BY。

### 关于 Order By

如果存在别名列，别名表，UNION 和 JOIN，Order By 的处理会变得很困难。MyBatis Dynamic SQL 库采取了一个简单的方法 —— 将列别名或者列名写入 Order By 子句，表别名（如果存在的话）将会被忽略。

:::important[Question]

> In our testing, this caused an issue in only one case. When there is an outer join and the select list contains both the left and right join column. In that case, the workaround is to supply a column alias for both columns.

这一句话理解不能...就把原文放在这里了...还望看客大佬们提点 (〃ﾉωﾉ)
:::

当使用函数（lower，upper 等）时，习惯上会给计算结果设置一个别名。这种情况下，可以使用 `sortColumn` 为 Order By 指定列名，如下所示：

```java
// select substring(employee_table.occupation, 1, 1) as ShortName, max(employee_table.birth_date) as MaxBirthday
// from employee employee_table
// group by substring(employee_table.occupation, 1, 1)
// order by MaxBirthday DESC
SelectStatementProvider selectStatement = select(substring(occupation, 1, 1).as("ShortName"), max(birthDate).as("MaxBirthday"))
    .from(employee, "employee_table")
    .groupBy(substring(occupation, 1, 1))
    .orderBy(sortColumn("MaxBirthday").descending())
    .build()
    .render(RenderingStrategy.MYBATIS3);
```

## DELETE

```java
DeleteStatementProvider deleteStatement = deleteFrom(employee)
    .where(occupation, isNull())
    .build()
    .render(RenderingStrategy.MYBATIS3);
```

## INSERT

```java
Employee record = new Employee();
record.setId(100);
record.setFirstName("Joe");
record.setLastName("Jones");
record.setBirthDate(LocalDate.now());
record.setEmployed("foo");
record.setOccupation("Developer");

InsertStatementProvider insertStatement = insert(record)
    .into(employee)
    .map(id).toProperty("id")
    .map(firstName).toProperty("firstName")
    .map(lastName).toProperty("lastName")
    .map(birthDate).toProperty("birthDate")
    .map(employed).toProperty("employed")
    .map(occupation).toProperty("occupation")
    .build()
    .render(RenderingStrategy.MYBATIS3);

int rows = mapper.insert(insertStatement);
```

注意 `map` 方法，它用于将数据库列映射到要插入记录的属性。有这几种不同的映射方法可用：

1. `map(column).toNull()` 将插入 null
2. `map(column).toConstant(constant_value)` 将插入常量（直接写入生成的 INSERT 语句中）
3. `map(column).toStringConstant(constant_value)` 将插入常量（被单引号 `'` 包裹）
4. `map(column).toProperty(property)` 将插入 record 的属性值
5. `map(column).toPropertyWhenPresent(property, Supplier<?> valueSupplier)` 仅当 record 的属性值不是 null 时插入，这被用来生成 selective 方法

## UPDATE

```java
UpdateStatementProvider updateStatement = update(employee)
    .set(firstName).equalTo("Alice")
    .set(lastName).equalToWhenPresent(record::getLastName)
    .where(id, isIn(1, 5, 7))
    .or(id, isIn(2, 6, 8), and(occupation, isLike("%bat")))
    .or(id, isGreaterThan(60))
    .and(birthDate, isBetween(LocalDate.of(1990, 1, 1)).and(LocalDate.now()))
    .build()
    .render(RenderingStrategy.MYBATIS3);

int rows = mapper.update(updateStatement);
```

注意 `set` 方法，它用于设置数据库列的值。它的几种变体拥有与上述 `map` 方法相似的性质：

1. `set(column).equalToNull()`
2. `set(column).equalToConstant(String constant)`
3. `set(column).equalToStringConstant(String constant)`
4. `set(column).equalTo(T value)`
5. `set(column).equalTo(Supplier<T> valueSupplier)`
6. `set(column).equalToWhenPresent(T value)`
7. `set(column).equalToWhenPresent(Supplier<T> valueSupplier)`

## XML mapper

虽然不推荐，但你仍然可以使用 xml mapper，它看起应该是这样的：

```java
@Mapper
public interface EmployeeMapper {
  int update(UpdateStatementProvider updateStatement);
}
```

xml 文件应该是这样：

```xml
<update id="update">
  ${updateStatement}
</update>
```

## Rendering Strategies

RenderingStrategy 决定生成 SQL 参数占位符的格式，目前内置了两种策略：

1. `RenderingStrategy.MYBATIS3`：适配 MyBatis3，生成 `#{param,jdbcType=INTEGER}`
2. `RenderingStrategy.SPRING_NAMED_PARAMETER`：适配 Spring NamedParameterJDBCTemplate， 生成 `:param`

## 针对 MyBatis3 的特别支援

```java
default QueryExpressionDSL<MyBatis3SelectModelAdapter<List<Employee>>> selectByExample() {
  return SelectDSL.selectWithMapper(this::selectMany, id, firstName, lastName, birthDate, employed, occupation)
      .from(employee);
}
```

正如前文 MyBatis Generator 生成的诸多 ByExample 方法，这些方法仅包括一些样板式代码，它们返回一个还在构造中的语句，调用者可以可选的任意定制 WHRER 子句。

# 附 I：SqlBuilder {#appendix-01}

|            COMMENT            |                                                                                        SQL                                                                                         |
| :---------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|          statements           |                                                `deleteFrom`, `insert`, `insertInto`, `select`, `selectDistinct`, `update`, `where`                                                 |
|  where condition connectors   |                                                                                    `or`, `and`                                                                                     |
|         join support          |                                                                                  `and`, `equalTo`                                                                                  |
|       aggregate support       |                                                                        `count`, `max`, `min`, `avg`, `sum`                                                                         |
|           functions           |                                                                        `add`, `lower`, `substring`, `upper`                                                                        |
| conditions for all data types | `isNull`, `isNotNull`, `isEqualTo`, `isNotEqualTo`, `isGreaterThan`, `isGreaterThanOrEqualTo`, `isLessThan`, `isLessThanOrEqualTo`, `isIn`, `isNotIn`, `isBetween`, `isNotBetween` |
|  conditions for strings only  |                            `isLike`, `isLikeCaseInsensitive`, `isNotLike`, `isNotLikeCaseInsensitive`, `isInCaseInsensitive`, `isNotInCaseInsensitive`                             |
|       order by support        |                                                                                    `sortColumn`                                                                                    |

# Reference

- [MyBatis Dynamic SQL](http://www.mybatis.org/mybatis-dynamic-sql/docs/introduction.html)
- [MyBatis Generator Core - MyBatis Dynamic SQL Usage Notes](http://www.mybatis.org/generator/generatedobjects/dynamicSql.html)
- [mybatis/mybatis-dynamic-sql](https://github.com/mybatis/mybatis-dynamic-sql "github repo")
- [mybatis/generator](https://github.com/mybatis/generator "github repo")
