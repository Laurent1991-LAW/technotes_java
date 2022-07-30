# 深度解析resultMap标签



一般来说，**一张表对应一份mapper映射文件，但部分属性在数据库中的存储并非直接存储其内容，而是存储其唯一标识**，比如id，如下表中的员工政治面貌、部门、职级、岗位均是通过各个分表的id进行维护：

![Snipaste_2022-05-18_00-05-47](C:\Users\Asus\Desktop\resultMap\images\Snipaste_2022-05-18_00-05-47.png)

<br/>

因此，我们可以在Mybatis的XML文件中，用resultMap标签，对**数据库中的所有字段与Java实体类的各个字段**进行一一对应：

```xml
<resultMap id="BaseResultMap" type="org.javaboy.vhr.model.Employee">
        <id column="id" property="id" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <result column="gender" property="gender" jdbcType="CHAR"/>
        <result column="birthday" property="birthday" jdbcType="DATE"/>
        <result column="idCard" property="idCard" jdbcType="CHAR"/>
        <result column="wedlock" property="wedlock" jdbcType="CHAR"/>
        <result column="nationId" property="nationId" jdbcType="INTEGER"/>
        <result column="nativePlace" property="nativePlace" jdbcType="VARCHAR"/>
        <result column="politicId" property="politicId" jdbcType="INTEGER"/>
        <result column="email" property="email" jdbcType="VARCHAR"/>
        <result column="phone" property="phone" jdbcType="VARCHAR"/>
        <result column="address" property="address" jdbcType="VARCHAR"/>
        <result column="departmentId" property="departmentId" jdbcType="INTEGER"/>
        <result column="jobLevelId" property="jobLevelId" jdbcType="INTEGER"/>
        <result column="posId" property="posId" jdbcType="INTEGER"/>
        <result column="engageForm" property="engageForm" jdbcType="VARCHAR"/>
        <result column="tiptopDegree" property="tiptopDegree" jdbcType="CHAR"/>
        <result column="specialty" property="specialty" jdbcType="VARCHAR"/>
        <result column="school" property="school" jdbcType="VARCHAR"/>
        <result column="beginDate" property="beginDate" jdbcType="DATE"/>
        <result column="workState" property="workState" jdbcType="CHAR"/>
        <result column="workID" property="workID" jdbcType="CHAR"/>
        <result column="contractTerm" property="contractTerm" jdbcType="DOUBLE"/>
        <result column="conversionTime" property="conversionTime" jdbcType="DATE"/>
        <result column="notWorkDate" property="notWorkDate" jdbcType="DATE"/>
        <result column="beginContract" property="beginContract" jdbcType="DATE"/>
        <result column="endContract" property="endContract" jdbcType="DATE"/>
        <result column="workAge" property="workAge" jdbcType="INTEGER"/>
    </resultMap>
```

<br/>

数据库通过id进行主表与分表之间的关系维护，在实际应用中，我们也**不会孤零零地只应用一张表的内容，而是经常会进行多表联查，后者让表与表之间产生关联**，而我们若想让程序体现这些关联，不是单纯地建一个主类（比如员工类）、一个分类（比如部门类）就完事儿了，我们会**在主体类的内部注入分类**，比如员工类里会包含部门类、职级类、民族类等等：

```java
public class Employee {
    // 对应数据库中主表存储的字段
    private Integer nationId;
    private Integer politicId;
    private Integer departmentId;
    private Integer jobLevelId;
    private Integer posId;
    // 对应数据库中主表依赖分表id所对应的各个分表
    private Nation nation;
    private Politicsstatus politicsstatus;
    private Department department;
    private JobLevel jobLevel;
    private Position position;
    // ...
}
```

<br>

这时候问题来了，我们在上面BaseResultMap中，仅仅对主表出现的字段进行了映射，也就是说，只有对应分表的id们和Java主体类的属性实现了映射（Employee类的上部分），但对象类字段——比如下部分的民族对象、政治面貌对象、部门对象等——还没有和所对应数据库的分表作映射呢！

如何实现这个映射？—— 在另一个resultMap标签中完成主表对象与关联分表对象的映射，若该分表对象与主表为一对一关系（比如员工与部门，一个员工对应一个部门），则内部使用association标签；若该分表对象与主表为一对多关系（比如教师与课程，一个教师对应多个课程），则内部使用collection标签。

```xml
<resultMap id="AllEmployeeInfo" type="org.javaboy.vhr.model.Employee" extends="BaseResultMap">
        <association property="nation" javaType="org.javaboy.vhr.model.Nation">
            <id column="nid" property="id"/>
            <result column="nname" property="name"/>
        </association>
        <association property="politicsstatus" javaType="org.javaboy.vhr.model.Politicsstatus">
            <id column="pid" property="id"/>
            <result column="pname" property="name"/>
        </association>
        <association property="department" javaType="org.javaboy.vhr.model.Department">
            <id column="did" property="id"/>
            <result column="dname" property="name"/>
        </association>
        <association property="jobLevel" javaType="org.javaboy.vhr.model.JobLevel">
            <id column="jid" property="id"/>
            <result column="jname" property="name"/>
        </association>
        <association property="position" javaType="org.javaboy.vhr.model.Position">
            <id column="posid" property="id"/>
            <result column="posname" property="name"/>
        </association>
    </resultMap>
```

<br>

**注意事项：**

1. association和collection标签里的property属性对应主类中该对象的属性名，该类的路径前者用javaType属性表示，后者用ofType属性；
2. association和collection标签内部的id/result标签的**column属性不一定与数据库表字段一致**！而是与sql语句查询出的结果字段一致（比如利用AS关键字可以新定义查询出来的字段名），**column与查询结果字段一致，切记切记**。

