---
layout: post
comments: true
title: "ibatis进阶(旧)"
description: "以前写的关于ibatis的内容"
categories: ["实践", "ibatis"]
---

## Ibatis比较少人使用的配置语法
简单来说，ibatis3虽然没有ognl,不过也支持基本的表达式（看起来有点像el表达式的样子) 上次有个问题，说到两个表单数据(两个javabean)，入同一个表，传参就应该不成问题了

#### java代码1:
{% highlight java %}
   Map map = new HashMap();
   ComplexBean bean = new ComplexBean();
   bean.setMap(new HashMap());
   bean.getMap().put("id", new Integer(1));
   map.put("bean", bean);
   Account account = new Account();
   account.setId(2);
   Account anotherAccount = new Account();
   anotherAccount.setId(3);
   map.put("accounts", new Account[] {account, anotherAccount});
   Integer id = (Integer) sqlMap.queryForObject("mapBeanMap", map);
{% endhighlight %}

#### ibatis配置1:
{% highlight xml %}
 <select id="mapBeanMap"
   parameterClass="map"
   resultClass="int" >
   select count(ACC_ID) from Account where ACC_ID in (#bean.map.id#,#accounts[0].id#,#accounts[1].id#)
 </select>
{% endhighlight %}

#### java代码2:
{% highlight java %}
   Map map = new HashMap();
   ComplexBean bean = new ComplexBean();
   bean.setMap(new HashMap());
   Account account = new Account();
   account.setId(2);
   Account anotherAccount = new Account();
   anotherAccount.setId(3);
   bean.getMap().put("accounts", new Account[] {account, anotherAccount});
   map.put("bean", bean);
{% endhighlight %}

#### ibatis配置2:
{% highlight xml %}
 <select id="mapBeanMap2"
   parameterClass="map"
   resultClass="int" >
   select count(ACC_ID) from Account where ACC_ID in
   <iterate close=")" open="(" conjunction="," property="bean.map.accounts">
     #bean.map.accounts[].id#
   </iterate>
 </select>
{% endhighlight %}

## ibatis与泛型
当使用复杂配置并且参数带有泛型的时候，使用比较标签有可能导致如下错误:There is no READABLE property named ‘XXX’ in class ‘java.lang.Object’.这是因为进行比较的时候，ibatis是通过反射获取类型而不是先计算值的,这样泛型的时候会获取到Object类而不能得到真实的类型,自己简单打个补丁先:
{% highlight diff %}
Index: src/com/ibatis/sqlmap/engine/mapping/sql/dynamic/elements/ConditionalTagHandler.java
===================================================================
--- src/com/ibatis/sqlmap/engine/mapping/sql/dynamic/elements/ConditionalTagHandler.java (revision 1079874)
+++ src/com/ibatis/sqlmap/engine/mapping/sql/dynamic/elements/ConditionalTagHandler.java (working copy)
@@ -72,14 +72,13 @@

     if (prop != null) {
       value1 = PROBE.getObject(parameterObject, prop);
-      type = PROBE.getPropertyTypeForGetter(parameterObject, prop);
     } else {
       value1 = parameterObject;
-      if (value1 != null) {
-        type = parameterObject.getClass();
-      } else {
-        type = Object.class;
-      }
+    }
+    if (value1 != null) {
+     type = value1.getClass();
+    } else {
+     type = Object.class;
     }
     if (comparePropertyName != null) {
       Object value2 = PROBE.getObject(parameterObject, comparePropertyName);
{% endhighlight %}

## 关于inlineParameterMap
例如#name#(标准配置),#name:NUMBER#(以:分割),#myVar:javaType=int#都是有效的
其中以:分割的有两种方式，#name:jdbcTypeName#,#name:jdbcTypeName:nullvalue#(如果后面还有则会被加到nullvalue上去) 这是老配置方法，个人不推荐使用。
最后一种是新的配置方式,可以带上javaType,jdbcType,mode,nullValue,numericScale,handler等参数(这个文档有详细描述)

## jdbcType,javaType和TypeHandler
首先要说明一点的是,配置里边的jdbcType和javaType两个配置参数是为了生成TypeHandler(如果没有指定的话);
查找typeHandler的内部结构是Map&lt;javaType, Map&lt;jdbcType, typeHandler&gt;&gt;,其中javaType是一个类,jdbcType是一个字符串;
所以jdbcType其实和数据库的字段类型没什么关系,只要能找到相应的TypeHandler即可(当然通常都会对应上);
typeHandler主要是做什么用的呢?无非就是使用jdbc api的时候选择setString/setInt还是getString/getObject之类

## 只指定resultClass，没有resultMap
如果没有指定resultMap，ibatis会根据parameterClass生成一个AutoResultMap对象;
对于AutoResultMap,里边的每个属性的映射对应的typeHandler是什么?
 
<table markdown="1" class="table">
  <tr><td>resultClass</td><td>TypeHandler</td></tr>
  <tr><td>Map</td><td>ObjectTypeHandler</td></tr>
  <tr><td>原型类型</td><td>相应类对应的typeHandler(javaType=?,jdbcType=null)</td></tr>
  <tr><td>Bean</td><td>会对实例变量名称进行大写并和ResultSetMetaData信息进行对比,最后生成typeHandler(javaType=?,jdbcType=null)</td></tr>
</table>
所以使用parameterClass是map的时候，某些字段的处理可能会有点问题,例如oracle的NUMBER类型会被转成BigDecimal类;

## 只指定parameterClass，没有parameterMap
如果没有指定parameterMap，就会根据配置的sql解析inlineParameterMap;
其中每个参数的TypeHandler如果没有指定，会根据参数的类型来寻找，例如#name,jdbcType=NUMBER# 会根据name计算后的类型来制定javaType
这个typeHandler的好处可以对jdbc api友好，例如对于int默认会采用IntegerTypeHandler，这样会调用PreparedStatement#setInt, 而不是统统setString或者setObject。
通常参数类型和jdbc类型不对应的时候，需要考虑设置typeHandler或者使用更强类型的Bean而不是统统使用map;

