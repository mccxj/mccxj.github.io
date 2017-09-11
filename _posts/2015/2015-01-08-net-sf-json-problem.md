---
layout: post
comments: true
title: "使用net.sf.json库进行json反序列化时存在的问题"
description: "net.sf.json使用问题"
categories: ["json"]
---

### 问题描述

```java
String content = "{\"response_head\":{\"menuid\":\"xxx\",\"process_code\":\"xxx\",\"verify_code\":\"\",\"resp_time\":\"20150107103234\",\"sequence\":{\"resp_seq\":\"20150107103301\",\"operation_seq\":\"\"},\"retinfo\":{\"retcode\":\"120\",\"rettype\":\"0\",\"retmsg\":\"[182096|]处理失败,原因:[屏蔽具体的失败原因！]\"}},\"response_body\":{} }";
JSONObject object = JSONObject.fromObject(content);
System.out.println(object.toString());

/*
{"response_head":{"menuid":"xxx","process_code":"xxx","verify_code":"","resp_time":"20150107103234","sequence":{"resp_seq":"20150107103301","operation_seq":""},"retinfo":{"retcode":"120","rettype":"0","retmsg":["182096|"]}},"response_body":{}}
*/
```

### 问题分析

采用json-lib-2.4-jdk15.jar，测试代码如上，会发现retmsg的值变成"[182096|".

测试简化json字符串，最终效果如下：

解析失败的例子：
```json
"{\"response_head\":{\"retmsg\":\"[182096|]处理失败,原因:[屏蔽具体的失败原因！]\"}}"
```

继续简化的话，就会解析成功
```json
"{\"response_head\":\"[182096|]处理失败,原因:[屏蔽具体的失败原因！]\"}"
```

找了一下源码，发现json-lib在某些情况下(绕来绕去，断点发现的)会尝试解析字符串，看看是不是json对象。（尼玛，太智能了）

AbstractJSON.java中的260行,这个时候str是后面的内容。
```java
         } else if( JSONUtils.mayBeJSON( str ) ) {
            try {
               return JSONSerializer.toJSON( str, jsonConfig );
            } catch( JSONException jsone ) {
               return str;
            }
         }
```
         
JsonArray.java中的1130行，这个时候v已经是"182096|"。这个时候会判断v是不是一个json对象，如果搞一个数组回去，否则就是搞一个字符串(上述现象)。
```java
               tokener.back();
               Object v = tokener.nextValue( jsonConfig );
               if( !JSONUtils.isFunctionHeader( v ) ){
                  if( v instanceof String && JSONUtils.mayBeJSON( (String) v ) ){
                     jsonArray.addValue( JSONUtils.DOUBLE_QUOTE + v + JSONUtils.DOUBLE_QUOTE,
                           jsonConfig );
                  }else{
                     jsonArray.addValue( v, jsonConfig );
                  }
                  fireElementAddedEvent( index, jsonArray.get( index++ ), jsonConfig );
               }
```

例如，下面的情况会产生一个数组：
```json
"{\"response_head\":{\"retmsg\":\"[{1820: 96|}]处理失败,原因:[屏蔽具体的失败原因！]\"}}"

{"response_head":{"retmsg":[{"1820":"96|"}]}}
```

关于如何判断是否是json,是会判断以[开头，以]结束的，刚好中枪。而尝试去截取中间内容的时候，又碰巧遇到中间的]字符，所以生成的字符串就是被截断了一部分的。
```java    
   /**
    * Tests if the String possibly represents a valid JSON String.<br>
    * Valid JSON strings are:
    * <ul>
    * <li>"null"</li>
    * <li>starts with "[" and ends with "]"</li>
    * <li>starts with "{" and ends with "}"</li>
    * </ul>
    */
   public static boolean mayBeJSON( String string ) {
      return string != null
            && ("null".equals( string )
                  || (string.startsWith( "[" ) && string.endsWith( "]" )) || (string.startsWith( "{" ) && string.endsWith( "}" )));
   }
```

### 问题结论

* 当json对象中某个值是以"{"开头，"}"结束，或者"["开头,"]"结束的时候，解析结果可能不是期望的。
* 不幸的是，目前来看，这个问题是无解的，考虑使用其他json库吧。