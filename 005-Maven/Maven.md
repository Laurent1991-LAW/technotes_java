# Maven高级应用





# 一、基本指令

- compile：编译成功，本模块即可启动；
- test：自动运行所有单元测试，最后提示成功、失败、跳过数量；
- install：将本模块载入本地仓库，以便其他项目可以引入本资源；



# 二、聚合与继承



## 概述

- 前提：在循环依赖的前提下，dao引入pojo资源、service引入pojo资源、controller引入service资源，即可保证各模块获取需要的组件；
- 缺陷：若修改dao文件，其他模块无法通信获取更新结果；
- 解决：利用一个父模块root，统一管理各个模块的依赖；

![Snipaste_2022-09-12_11-04-55](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_11-04-55.png)



## 聚合步骤

1. 建立一个maven父模块，内部只需要一个pom文件，src目录可删除；
2. 添加packaging标签，打包方式为pom；
3. 添加modules标签，内部包含各个子模块；
4. 直接compile或install父模块，即可完成所有子模块的资源导入；

![Snipaste_2022-09-12_13-48-54](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_13-48-54.png)

**备注1：**子模块打包方式默认为jar，web模块打包方式为war，在创建maven工程时即需可定义；

**备注2**：各个模块的编译顺序与各个模块之间的依赖关系有关，最深层依赖的jar最先编译 —— 此规则适用于线性依赖关系，攀枝错节的依赖关系需要配置来解决加载顺序；

![Snipaste_2022-09-12_14-08-37](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-08-37.png)

## 继承步骤

继承指依赖的父子模块继承关系

![Snipaste_2022-09-12_14-18-04](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-18-04.png)



1. 父工程使用\<DependencyManagement>标签来管理所有需要被子工程继承的依赖；
2. 子工程需要在pom文件中添加Parent标签，可相伴随如下图修改；
3. 至此，子工程中的依赖版本一律由父工程的\<DependencyManagement>进行定义，子工程中依赖的version标签全部可删除；

![Snipaste_2022-09-12_14-24-31](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-24-31.png)



# 三、属性定义与使用

**类型包括：**

1. 自定义属性
2. 内置属性
3. setting属性
4. java系统属性
5. 环境变量属性



## 自定义属性

**概述**：相当于定义一个POM文件内可用的变量，用 ${  } 进行引用；

![Snipaste_2022-09-12_14-49-31](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-49-31.png)





## 内置属性



![Snipaste_2022-09-12_14-52-17](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-52-17.png)

**备注**：version为当前模块的版本号，比如子模块版本需与父模块一致，\<DependencyManagement>中的子模块版本就可写为${project.version}



## Settings属性

读取Maven的settings配置文件内容

![Snipaste_2022-09-12_14-52-42](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-52-42.png)



## Java系统属性

![Snipaste_2022-09-12_14-53-09](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-53-09.png)



## 环境变量属性

![Snipaste_2022-09-12_14-55-29](E:\doc_repo\005-Maven\images\Snipaste_2022-09-12_14-55-29.png)







