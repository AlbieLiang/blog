---
layout: post
title: ArbitraryGen之混合模式（使用）
tagline:
category: tools
tags : [tools, android, java, arbitrarygen, code generator, hybrid mode, template]
---
{% include JB/setup %}
本文章将介绍代码生成器ArbitraryGen的混合模式（hybrid mode）。如果你还不是很了解什么是ArbitraryGen，请移步ArbitraryGen首页，上面会有ArbitraryGen的简要介绍。

（[点击到ArbitraryGen首页]({{ site.baseurl }}/tools/2016/07/22/arbitrarygen-home-page)）

在介绍使用之前，先看下混合模式的特点

### 混合式代码生成器具有的特性：

（1）将脚本和源码混合到代码的源文件中，并且脚本不影响源码的编译等；

（2）代码生成操作是可持续的；

（3）脚本在代码生成后不会被删除，而是将生成的代码附于脚本区域之后，以便于脚本的可持续使用；

（4）每次生成代码前会将上次生成的代码删除；


#### 下面就进入本文的主题了：混合式代码生成器的使用


首先我们需要[引入ArbitraryGen]({{ site.baseurl }}/tools/2016/07/22/arbitrarygen-home-page#user-content-引入ArbitraryGen)到项目当中，详细操作可以移步ArbitraryGen首页，在此不再冗述。


### 使用方法

使用方法其实很简单，其包括（1）数据源的定义；（2）模板文件；（3）工程中如何引入使用；

#### 1、数据源：
数据源是以.hybrids-define为后缀的文件，文件格式是XML，其中二级节点作为数据源的元数据；
（数据源文件的位置是通过配置想决定的，可以参考[ArbitraryGen首页]({{ site.baseurl }}/tools/2016/07/22/arbitrarygen-home-page)）

示例如下：

    <?xml version="1.0" encoding="UTF-8"?>
    <hybrids-define
    	package="com.flyfox.player.autogen.table"
    	delegateDest="src/com/flyfox/player/autogen/table"
    	delegatePkg="com.flyfox.player.autogen.table"
    	delegate="VDBInfoDelegate"
    	delegateSuffix="java"
    	tag="table">

    	<!--
    	    * 根节点属性：
    	    *
    	    * package        : 子节点用于生成代码的包名
    	    * delegate       : 委派模板文件名（收敛子节点的统一文件）
    	    * delegateDest   : 委派模板文件所在的根目录（注：该模式下模板和生成的文件是同一个）
    	    * delegatePkg    : 委派模板文件的包名
    	    * delegateSuffix : 委派模板文件的文件后缀（为了支持不同语言）
    	    * tag            : 作为委派模板数据源的子节点集合，“,”作为分隔符，tag与模板文件的映射关系可在template-libs/template-mapping.properties中指定
    	    *
    	-->

    	<!--
    		* acceptable type for fields:
    		*
    		* boolean <==> INT(0/1) <br />
    		* int/Integer <==> INT <br/>
    		* long/Long <==> LONG <br/>
    		* short/Short <==> SHORT <br/>
    		* float/Float <==> FLOAT <br/>
    		* double/Double <==> DOUBLE <br/>
    		* String <==> TEXT <br/>
    		* byte[] <==> BLOB <br/>
    		* protobuf / referto <==> BLOB<br/>
    	-->


    	<!-- The rowId of this table will be use as primaryKey. -->
    	<table name="DBItem_1">
    		<field name="field_1" type="String" comment="for duplicate removal"/>
    		<field name="field_2" type="String"/>
    		<field name="field_3" type="byte[]">
    			<part name="field_part_1" type="long"/>
     		</field>
    		<field name="field_4" type="boolean" default="false" index="Table_1_index”/>

    		<index name="field_3_and_1_index" fields="field_3,field_1"/>
    	</table>

    	<!-- The vigor entry will not be created for this table because noEntry is true. -->
    	<table name="DBItem_2" noEntry="true">
    		<field name="field_1" type="String" primaryKey="1"/>
    		<field name="field_2_2" type="String"/>
    		<field name="field_2_3" type="String"/>
    		<field name="field_2_4" type="byte[]">
    			<part name="field_part_1" type="long"/>
     		</field>
    	</table>

    	<table name="DBItem_3">
    		<field name="field_3_1" type="String" primaryKey="1"/>
    		<field name="field_3_2" type="String"/>
    		<field name="field_3_3" type="byte[]">
    			<part name="field_part_1" type="long"/>
     		</field>
    	</table>

    	<table name="DBItem_4">
    		<field name="field_4_1" type="String" primaryKey="1"/>
    		<field name="field_4_2" type="String"/>
    	</table>

    	<!-- Multiply primary key sample -->
    	<table name="DBItem_5">
    		<field name="field_5_1" type="String" primaryKey="1"/>
    		<field name="field_5_2" type="int" primaryKey="2"/>
    		<field name="field_5_3" type="long"/>
    	</table>
    </hybrids-define>


#### 2、代码模板

模板程序就是普通的java文件(可以是其它格式文件)，脚本被嵌入到源码中的/*@@@#SCRIPT-BEGIN# 和 #SCRIPT-END#@@@*/ 之中，则每个脚本区生成的代码将append到脚本区域后面，处于//@@@#AUTO-GEN-BEGIN# 和 //@@@#AUTO-GEN-END# 注释之间。示例如下：
{% raw %}

    package com.flyfox.player.autogen.table;

    import android.util.Log;

    import com.flyfox.sdk.db.base.IDatabaseEngine;
    import com.flyfox.sdk.db.base.IDatabaseInfoDelegate;

    /*@@@#SCRIPT-BEGIN#
    <%if (_tables && _tables.length > 0) {%>
    	<%for (var i = 0; i < _tables.length; i++) {%>
    import <%=_package%>.<%=_tables[i]._name%>;<%}%>
    <%}%>
    #SCRIPT-END#@@@*///@@@#AUTO-GEN-BEGIN#




    import com.flyfox.player.autogen.table.DBItem_1;
    import com.flyfox.player.autogen.table.DBItem_2;
    import com.flyfox.player.autogen.table.DBItem_3;
    import com.flyfox.player.autogen.table.DBItem_4;
    import com.flyfox.player.autogen.table.DBItem_5;


    //@@@#AUTO-GEN-END#
    /**
     * Generated by ScriptCodeGenEngine.
     * <p>
     * Auto generate add {@link IVigorDBInfo} into {@link IDatabaseEngine} here.
     *
     * @author AlbieLiang
     *
     */
    public class VDBInfoDelegate implements IDatabaseInfoDelegate {

       private static final String TAG = "AutoGen.VDBInfoDelegate";

       @Override
       public void delegate(IDatabaseEngine engine) {
          // Auto generate code here

          /*@@@#SCRIPT-BEGIN#
           <%if (_tables && _tables.length > 0) {%>
                <%for (var i = 0; i < _tables.length; i++) {%>
           engine.addDatabaseInfo(<%=_tables[i]._name%>.getVDBInfo());<%}%>
           <%}%>
           #SCRIPT-END#@@@*///@@@#AUTO-GEN-BEGIN#



          engine.addDatabaseInfo(DBItem_1.getVDBInfo());
          engine.addDatabaseInfo(DBItem_2.getVDBInfo());
          engine.addDatabaseInfo(DBItem_3.getVDBInfo());
          engine.addDatabaseInfo(DBItem_4.getVDBInfo());
          engine.addDatabaseInfo(DBItem_5.getVDBInfo());


    //@@@#AUTO-GEN-END#
       }
    }

{% endraw %}

##### 1）模板中使用到的主要数据数据来自于数据源，主要变量如下：

（1）变量_package，来源于根节点的package属性，改值将传递到子节点的处理程序当中；

（2）变量_name，来源于根节点的delegate属性；

（3）列表数据，来源于二级标签，其中脚本中变量名规则是：数据源中的二级标签名字前面加个下滑下，并在末尾加上一个”s"（如：示例中的_tables）；会有一种特殊的情况出现，就是标签的名字中包含“-”，这时候，会自动将“-”自动使用下划线“_”代替。

##### 2）元数据的处理：
看到这里你可能会觉得有哪些地方不太对的样子是吧，恩，没错，这些元数据都干什么用呢？其实在一开始的数据源文件中，已经有一段注释提示到了，就是根节点中的tag属性的作用，其映射了一个代码生成模板文件，用来处理这些元数据的，哈哈，映射关系需要写在template-libs/template-mapping.properties文件当中。如果没有tag属性或tag属性的value为空时，ArbitraryGen会自动读取根节点下的所有一级节点，并记录子节点的所有name作为tag属性值。

// todo 加上模板和生成的代码。


### 结语：
大家可能会吐槽为什么举的例子都是这么复杂的代码，而且很长，哈哈，莫急，这个其实是我写的另一套数据库框架，有空再写写文章，开源下这套东西，敬请期待。

其实这个代码生成器是一个不是简单的一套生成代码的小工具，它提供了部分可扩展的接口，提供SDK，若想是想更加强大功能的定制化的代码生成可以使用SDK进行开发，后续将陆续写上这一块的文章。代码生成器的实现，也将在后续文章中说明。敬请期待！
