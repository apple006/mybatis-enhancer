项目来由：
1，其实比较倾向于jpa的映射方式，当时jpa约束比较多，比较依赖经验说，团队人多的话，比较难把控，mybatis相对来说比较灵活，但是一大波内容配置在xml，尤其是项目初期，数据模型、表结构变数比较大，改配置文件就成了一个比较头痛的事情，幸好有了[abel533/Mapper](https://github.com/abel533/Mapper)项目，由于基础sql是动态生成的，这样表结构有变更时，也不用改一堆mapper文件，但是[abel533/Mapper](https://github.com/abel533/Mapper)有个问题，就是它是在mybatis之上封装的，如果想扩展的话，有2种方式：
1，基于[abel533/Mapper](https://github.com/abel533/Mapper)就要定制自己的Mapper接口，写自己的sqlProvider，这个工作量也不小
2，基于原生Mybatis，写自己的mapper.xml，问题就在这儿了，本来我是想用abel533/Mapper来省掉在mapper.xml配一堆字段名的，但是用mybaits原生的mapper.xml，还是要写死一大堆column，这样又走回老路子了。

由上：
研究了一下mybatis的源码，调整了一下依赖关系，mybatis依赖abel533/Mapper，在mybatis加载mapper.xml文件时，获取生成的动态sql，然后注册到上下文里，相当于mapper.xml映射文件默认有了一些动态sql，有需要用到时，直接在mapper.xml映射文件里include就行了；

话不多说，直接上代码：
```xml
<sql id="sql_cause">
	<trim prefix="where" prefixOverrides="and | or">
		<include refid="base_query_params" />
	</trim>
</sql>
<select id="findByParams" resultMap="BaseResultMap" parameterType="java.util.Map" >
	<include refid="select_column_list" />
	from system_setting
	<include refid="sql_cause" />
	<include refid="sql_restraint" />
</select>
<select id="countByParams" resultType="java.lang.Integer" parameterType="java.util.Map" >
	select count(*) from system_setting
	<include refid="sql_cause" />
</select>
```
注意：abel533/Mapper已经内置了大部分crud的方法，这个mapper.xml文件只是告诉大家怎么扩展，怎么让mybaits的mapper.xml映射文件也用上abel533/Mapper的利器

大家可能比较好奇？？ base_query_params、select_column_list、sql_cause、sql_restraint从哪儿来的，这个就是扩展了mybatis，动态生成的，内置的变量还有：
```xml
####resultMap
	<resultMap id="BaseResultMap" type="com.yao.heart.common.domain.Application">
	  <id column="id" property="id"/>
	  <result column="app_name" property="appName"/>
	  <result column="archived" property="archived"/>
	</resultMap>

####base_column_list
	id,app_name AS appName,archived

####base_query_params
  <if test="id != null">AND id = #{id,javaType=java.lang.Integer}</if>
  <if test="appName != null and appName != '' ">AND app_name = #{appName,javaType=java.lang.String}</if>
  <if test="archived != null">AND archived = #{archived,javaType=java.lang.Boolean}</if>

####select_count
	SELECT COUNT(id) 
	
####select_column_list
	SELECT id,app_name,archived

####update_column_list
	<set>
	  <if test="appName != null and appName != '' ">app_name = #{appName,javaType=java.lang.String},</if>
	  <if test="archived != null">archived = #{archived,javaType=java.lang.Boolean},</if>
	</set>

####sql_restraint	
<sql id="sql_restraint"><if test="orderBy != null">order by ${orderBy}</if><choose><when test="limit != null and limit gt 0 ">limit<choose><when test="offset != null and offset gt 0 ">#{offset},#{limit}</when><otherwise>0,#{limit}</otherwise></choose></when><otherwise>limit 0,100</otherwise></choose></sql>
```

备注：
1，为什么base_query_params不用`<where>`包一下？？
  因为有时候会写一些复杂的sql，放在外面，交给使用方自己去包一下更易于扩展。如上面mapper.xml那样用法。



MyBatis SQL Mapper Framework for Java
=====================================

[![Build Status](https://travis-ci.org/mybatis/mybatis-3.svg?branch=master)](https://travis-ci.org/mybatis/mybatis-3)
[![Coverage Status](https://coveralls.io/repos/mybatis/mybatis-3/badge.svg?branch=master&service=github)](https://coveralls.io/github/mybatis/mybatis-3?branch=master)
[![Dependency Status](https://www.versioneye.com/user/projects/56199c04a193340f320005d3/badge.svg?style=flat)](https://www.versioneye.com/user/projects/56199c04a193340f320005d3)
[![Maven central](https://maven-badges.herokuapp.com/maven-central/org.mybatis/mybatis/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.mybatis/mybatis)
[![License](http://img.shields.io/:license-apache-brightgreen.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)
[![Stack Overflow](http://img.shields.io/:stack%20overflow-mybatis-brightgreen.svg)](http://stackoverflow.com/questions/tagged/mybatis)
[![Project Stats](https://www.openhub.net/p/mybatis/widgets/project_thin_badge.gif)](https://www.openhub.net/p/mybatis)

![mybatis](http://mybatis.github.io/images/mybatis-logo.png)

The MyBatis SQL mapper framework makes it easier to use a relational database with object-oriented applications.
MyBatis couples objects with stored procedures or SQL statements using a XML descriptor or annotations.
Simplicity is the biggest advantage of the MyBatis data mapper over object relational mapping tools.

Essentials
----------

* [See the docs](http://mybatis.github.io/mybatis-3)
* [Download Latest](https://github.com/mybatis/mybatis-3/releases)
* [Download Snapshot](https://oss.sonatype.org/content/repositories/snapshots/org/mybatis/mybatis/)
