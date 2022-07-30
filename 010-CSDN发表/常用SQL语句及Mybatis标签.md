常用SQL语句及Mybatis标签



# 一、SQL语句



## case\when\then

**查询概要信息**

```sql
SELECT
a.process_node AS processNode,
i.project_name AS projectName,
i.project_manager AS projectManager,
i.creation_date AS creationDate,
c.contract_code AS contractCode,
s.partner AS partner,
CASE 
WHEN a.process_node = 1 then i.handler
WHEN a.process_node = 2 then s.handler
WHEN a.process_node = 3 then c.handler
ELSE ''
END AS handler
FROM main a 
LEFT JOIN initialization i ON a.charter_id = i.id
LEFT JOIN supplier_selection s ON a.supplier_selection_id = s.id
LEFT JOIN contract_review c ON a.contract_review_id = c.id
WHERE a.id = #{mainId}
```





## insert...on duplicate key update

```sql
// Mapper文件
int batchSaveOrUpateFiles(@Param("files") List<File> files)

// XML文件
<foreach collection="entities" item="item" seperator=";">
insert into file（
    id,
    fileName,
    creation_date,
    last_update_date
）
Values 
(
    #{item.id, jdbcType=bigint},
    #{item.fileName, jdbcType=varchar},
    sysdate(),
    sysdate()
)
on duplicate key update 
    fileName = #{item.fileName, jdbcType=varchar},
    last_update_date = sysdate()
</foreach>
```



## replace into ...

**场景：**

1. 员工工资账套对应表empsalary ； 
2. eid为员工ID号，创建唯一索引：UNIQUE KEY \`eid\` （\`eid `）；
3. replace into 可实现“若eid已存在则更新，否则即插入”的操作。

```sql
<insert id="updateEmployeeSalaryById">
        REPLACE INTO empsalary (eid,sid) VALUES(#{eid},#{sid})
</insert>
```





# 二、XML标签

## choose\when\otherwise标签

**中英文项目名重名判断**

```java
// Mapper 方法 
List<Project>  checkProjectNameExist(projectId, projectName, engFlag);

// SQL 语句
select * from XXX 
where 1 = 1
<choose>
    <when test='engFlag != null and engFlag = true'>
		and projectNameEn = #{projectName}
    </when>
    <when test='engFlag != null and engFlag = false'>
		and projectName = #{projectName}
    </when>
    <otherwise>
    	and 2 = 3
    </otherwise>
</choose>
<if test='projectId != null and projectId != ""'>
	and projectId != #{projectId}
</if>
```



## foreach标签

### 批量删除

```java
// 利用Stream获取List内id的List集合
List<File> list;
List<String> pids = list.stream().map(File::getProjectId).collect(Collectors.toList());

// Mapper文件
void batchDeleteFiles(@Param("ids") List<String> pids);
    
// 数据库中批量删除SQL语句
<delete parameterType="java.util.List">
    DELETE FROM XXX
    WHERE id IN
    <foreach collection ="ids" item="id" index="index" open="(" close=")" seperator=",">
            #{pids}
	</foreach>
</delete>
```

### 批量插入

```sql
insert into file（
    id,
    fileName,
    creation_date,
    last_update_date
）
Values 
<foreach collection="entities" item="item" open="(" close=")" seperator=";">
        #{item.id, jdbcType=bigint},
        #{item.fileName, jdbcType=varchar},
        sysdate(),
        sysdate()
</foreach>
```



## include标签

```sql
<!--查询实体类-->
<select id="getEmployeeByPage" resultMap="AllEmployeeInfo">
        SELECT 
            e.*,
            p.`id` AS pid,
            p.`name` AS pname,
            n.`id` AS nid,
            n.`name` AS nname,
            d.`id` AS did,
            d.`name` AS dname,
            j.`id` AS jid,
            j.`name` AS jname,
            pos.`id` AS posid,
            pos.`name` AS posname 
        FROM 
            employee e,
            nation n,
            politicsstatus p,
            department d,
            joblevel j,
            position pos 
        WHERE 
            e.`nationId`= n.`id` 
            AND e.`politicId`= p.`id` 
            AND e.`departmentId`=d.`id` 
            AND e.`jobLevelId`=j.`id` 
            AND e.`posId`=pos.`id`
    	<include refid="base_where">
        <if test="page != null and size!= null">
            LIMIT #{page},#{size}
        </if>
    </select>
   
<!--查询总个数--> 
    <select id="getTotal" resultType="java.lang.Long">
        SELECT count(*) FROM employee e
        <where>
            <include refid="base_where">
        </where>
    </select>

<sql id="where_base">
    <if test="emp!=null">
        <if test="emp.name !=null and emp.name!=''">
            AND e.name LIKE concat('%',#{emp.name},'%')
        </if>
        <if test="emp.politicId !=null">
            AND e.politicId = #{emp.politicId}
        </if>
        <if test="emp.nationId !=null">
            AND e.nationId = #{emp.nationId}
        </if>
        <if test="emp.jobLevelId !=null">
            AND e.jobLevelId = #{emp.jobLevelId}
        </if>
        <if test="emp.departmentId !=null">
            AND e.departmentId = #{emp.departmentId}
        </if>
        <if test="emp.engageForm !=null and emp.engageForm!=''">
            AND e.engageForm = #{emp.engageForm}
        </if>
        <if test="emp.posId !=null">
            AND e.posId = #{emp.posId}
        </if>
    </if>
    <if test="beginDateScope !=null">
        AND e.beginDate BETWEEN #{beginDateScope[0]} AND #{beginDateScope[1]}
    </if>
</sql>
```



## trim标签

### suffixOverrides

suffixOverrides属性：标签内部子语句的结尾未触suffix则保留，否则将被suffix复写

```xml
 <insert id="insertSelective" parameterType="org.javaboy.vhr.model.Employee" useGeneratedKeys="true" keyProperty="id">
        insert into employee
            <trim prefix="(" suffix=")" suffixOverrides=",">
                <if test="id != null">
                    id,
                </if>
                <if test="name != null">
                    name,
                </if>
                <if test="gender != null">
                    gender,
                </if>              
            </trim>
            <trim prefix="values (" suffix=")" suffixOverrides=",">
                <if test="id != null">
                    #{id,jdbcType=INTEGER},
                </if>
                <if test="name != null">
                    #{name,jdbcType=VARCHAR},
                </if>
                <if test="gender != null">
                    #{gender,jdbcType=CHAR},
                </if>                    
            </trim>
    </insert>
```



### prefixOverrides 

prefixOverrides属性：标签内部子语句的开头未触prefix则保留，否则将被prefix复写

```xml
<sql id = "base_where">
	<trim prefix="WHERE" prefixOverrides="AND |OR">
        <if test ='record.projectName != null and record.projectName != ""'>
            and project_name like concat("%", #{project_name},"%")
        </if>
        <if test ='record.partner != null and record.partner != ""'>
            and partner like concat("%", #{partner},"%")
        </if>
    </trim>
</sql>
```

