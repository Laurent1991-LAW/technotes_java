# Maven高级应用





# 一、概述



## 生命周期

### 概述

生命周期中靠后的指令执行时，会将它前面的操作都执行完毕，而它后方的指令则不会执行，比如执行install则必然会先compile与test

![Snipaste_2022-10-04_11-58-10](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-58-10.png)

### 跳过测试

**方式一**：跳过该工程下所有测试用例

![Snipaste_2022-10-04_12-01-40](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_12-01-40.png)



**方式二**：跳过该工程下所有测试用例

![Snipaste_2022-10-04_12-07-43](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_12-07-43.png)



**方式三**：跳过该工程下特定测试用例，依靠插件配置内的include/exclude标签，如

```xml
<plugin>
    <artifact>maven-surefire-plugin</artifact>
    <version>2.22.1</version>
    <configuration>
		<includes>
			<include>**/UserServiceTest.java</include>
		<\includes>
		<exclude>
			<exclude>**/ItemServiceTest.java</exclude>
		<\excludes>
	</configuration>
</plugin>
```



## 基本指令

- compile：编译成功，本模块即可启动；
- test：自动运行所有单元测试，最后提示成功、失败、跳过数量；
- install：将本模块载入本地仓库，以便其他项目可以引入本资源；



## 版本管理

![Snipaste_2022-10-04_10-58-36](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_10-58-36.png)

![Snipaste_2022-10-04_11-03-41](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-03-41.png)





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







# 四、POM文件管理资源属性

**功能**：利用POM文件对Resources目录下的配置文件属性进行统一管理，properties文件内用`${属性名}`的方式读取POM文件属性

## **步骤** 

1. 父工程POM文件properties标签中定义属性名与值；
2. 子工程properties文件中用`${属性名}`的方式获取该属性值；
3. 父工程POM文件在\<resources>标签中定义配置文件所在目录——Maven将根据该目录进行工程配置文件扫描，使用占位符的地方将进行对应属性的赋值；

![Snipaste_2022-10-04_11-20-12](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-20-12.png)

![Snipaste_2022-10-04_11-19-49](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-19-49.png)

![Snipaste_2022-10-04_11-12-21](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-12-21.png)

![Snipaste_2022-10-04_11-15-03](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-15-03.png)



## 标签配置

**备注** ：案例中的jdbc配置文件位于子工程ssm_dao中，父工程POM文件中的属性若想注入子工程的属性文件中，则需如下图进行目录地址配置。`../ssm_dao`意为退出到上一级目录，并进入ssm_dao目录下查找配置文件。

**改进**： 多个子工程不可能配置无数个resource标签，因此可改进如下`<directory> ${project.basedir}/src/main/resources </directory>`，之后将会扫描到所有子工程根目录下的配置文件。

![无标题](E:\doc_repo\005-Maven\images\无标题.png)



## 查验

如何查验properties配置文件中是否已经加载上pom文件中的值？

install以后，打开工程所在maven目录，打开打包完成后内部的jar包，点开properties文件看是否赋值成功。



# 五、多环境配置与启动

![Snipaste_2022-10-04_11-39-22](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-39-22.png)

**概要** ：不同的环境下对应不同的配置，该操作可在POM文件中利用profiles标签进行如下配置，注意不同的环境下有不同的properties标签；默认启动环境用activation标签标注；在maven打包命令中，用`install -p dev_env`进行对应环境的打包。

![Snipaste_2022-10-04_11-47-43](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-47-43.png)

![Snipaste_2022-10-04_11-48-47](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-48-47.png)

![Snipaste_2022-10-04_11-46-40](E:\doc_repo\005-Maven\images\Snipaste_2022-10-04_11-46-40.png)







