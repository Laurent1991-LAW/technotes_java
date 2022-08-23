# 一、首页



## 我的待办

请求路径 : /myTasks

请求方式 : GET

请求参数 : curPage, pageSize, orderBy

返回参数 : 

| 参数名 | 详情         |
| -----: | ------------ |
|   code | 响应码       |
|    msg | 响应信息     |
|   date | List<MyTask> |



## 我的项目

请求路径 : /myProjects

请求方式 : POST

请求参数 :

| pageInfo | queryInfo    |
| -------- | ------------ |
| curPage  | projectName  |
| pageSize | handler      |
| orderBy  | coopType     |
|          | partner      |
|          | contractCode |



```java
// controller层接收Model
public class ProjectVO {
    private PageInfo pageInfo;
    private ProjectModel record;
}

// xml
<select id = "selectAllProjects">
	<!-- 无高级搜索条件的sql语句 -->
	<include refid="base_where">
	<!-- 排序及分页逻辑 -->
</select>

<sql id = "base_where">
	<trim prefix="WHERE" prefixoverride="AND |OR">
        <if test ='record.projectName != null and record.projectName != ""'>
            and project_name like concat("%", TRIM(#{project_name}),"%")
        </if>
        <if test ='record.partner != null and record.partner != ""'>
            and partner like concat("%", TRIM(#{partner}),"%")
        </if>
    </trim>
</sql>
```

# 二、Charter节点

## 保存 / 提交

**请求路径 :** /save

**请求方式 :** POST

**请求参数 :** JSONArray

**返回参数 :**  id

**保存与提交 :** 创建人未提交前，submit字段为0，此时其他项目参与人无法在"我的项目/待办"中获取项目信息



**存入附件逻辑 :**

1. 传入list为null或为空，删除该projectId下所有附件 —— 用户删除了所有附件 ;
2. 利用Stream API获取List的ids，批量删除数据库中的附件  ;

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

3. 遍历list，id为null则新建 : 获取uuid —— setId —— setLastUpdatedBy —— setCreatedBy ;
4. id不为null则为更新 : setLastUpdatedBy ;

```java
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



## 采购代表确认 / 驳回 / 转他人处理

请求路径 : /procurementOper

请求方式 : POST

请求参数 : 

|        | 字段名   | 详情                         |
| ------ | -------- | ---------------------------- |
| 操作   | act      | 1-确认; 2-驳回; 3-转他人处理 |
| 意见   | comment  | 意见                         |
| 目标人 | destUuid | 转审目标人uuid               |
| 主表ID | mainId   |                              |



## 中英项目名重名判断

请求路径 : /checkNameExist

请求方式 : GET

请求参数 : projectId, projectName, engFlag



SQL语句（返回名字相同且id不同的集合）

```sql
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



## 查询概要信息

**请求路径 :** /findGeneInfo

**请求方式 :** GET

**请求参数 :** mainId

**返回参数 :** 

| 参数名称   | 参数名称       | 表名                                       |
| ---------- | -------------- | ------------------------------------------ |
| 项目名称   | projectName    | initialization                             |
| 项目经理   | projectManager | initialization                             |
| 创建时间   | creationDate   | initialization                             |
| 当前处理人 | handler        | charter/contract_review/supplier_selection |
| 合同编码   | contractCode   | contract_review                            |
| 合作方     | partner        | supplier_selection                         |
| 当前阶段   | processNode    | main                                       |

```sql lite
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



# 三、OA概念



## 会签

# 

1.    什么是会签
  a)    在流程业务管理中，任务是通常都是由一个人去处理的，而多个人同时处理一个任务，这种任务我们称之为会签任务。
2.    会签的种类
  a)    按数量通过：达到一定数量的通过表决后，会签通过。
  b)    按比例通过：达到一定比例的通过表决后，会签通过。
  c)    一票否决：只要有一个表决时否定的，会签通过。
  d)    一票通过：只要有一个表决通过的，会签通过。

Activiti实现会签
a)    Activiti实现会签是基于多实例任务，将节点设置成多实例，主要通过在UserTask节点的属性上配置









