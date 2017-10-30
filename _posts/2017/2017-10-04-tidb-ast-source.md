---
layout: post
comments: true
title: "tidb源码学习之ast包"
description: "对tidb中ast包的源码学习记录"
categories: ["tidb", "ast", "学习", "源码"]
---

## 基本结构 ast.go

![基本结构](http://www.plantuml.com/plantuml/png/TL6nJiCm49thh_1eLvG9CR4WbR000q5T44D8BfM5OqVnfIeg_NUSSsASe6Pmpk_UkzDxLWQXguiI-8kjW9yOzzzzMKABui1toYcqdUJ23Ds1SiNj5_-qLaksUeCZ2iaTTihisIe790JzCOAIdPcAAnwERJUk8V9t2m9RlaPVkEjCWQu6p4z-7BloNvEKkqBt80w5vd7uwHnaeINJ1acs1VQDg8QJXv64155encLq9LMcYxse_S5xF_3s9j09bICqSvZrffmSrhv6PStW6fppbPZ7aME3IUeE2uG632ves-tJDBAgT7w9zVp7QXQiXFOAhjVedKraRoTWbB3pjju_rWb2CSmODVqMD5f3C-z-DfluqYm-ES5JF2brIe75E0WUNI_Hu3BLpnpz0W00)

## 基本实现 base.go

和上图很类似，只是属于内部实现，提供了基本的能力，用于更细化的类型实现。
![基本实现](http://www.plantuml.com/plantuml/png/Iyv9B2vMoCjFILMevk8iIQqeKIWkAShCI-UgvKe6owLM51Jv0UMXtBJIl6GaRd59RWaIDoKb1vcNYymhIYqkpIa9JWMh1za64N3BJCr9ALQ8ZjKAGl21jdE17MLJewkBS0AC0H66EmL9ATmzC0P46EOkD56e-v3qepWI0000)

## 函数调用 functions.go
分别为聚合函数，普通函数，转换函数

![函数调用](http://www.plantuml.com/plantuml/png/IolDI_RBJqbLiAdHrLLmJ4ylIarFB4br0mgxLXGKSQMXo8E4dHDpSd1A5PU0f000)

## 可视化AST

为了方便理解AST，需要有个简单的可视化工具。
```
type astViewer struct {
	level int
}

// Enter implements ast.Visitor interface.
func (av *astViewer) Enter(inNode ast.Node) (outNode ast.Node, skipChildren bool) {
	strings.Repeat("  ", av.level)
	fmt.Println(reflect.TypeOf(inNode))
	av.level++
	return inNode, false
}

// Leave implements ast.Visitor interface.
func (av *astViewer) Leave(inNode ast.Node) (node ast.Node, ok bool) {
	av.level--
	return inNode, true
}

// How to use it.
node.Accept(&astViewer{})
```

## DML语句 dml.go

dml语句类型

![dml语句类型](http://www.plantuml.com/plantuml/png/Iyv9B2vMS4dDIIr93Ix9BL6miL580VDUh5_xj7-fWfqTLqfkZbz-Igg2JOskBf9IhcImNi-yujIY4fZUJ30FXrw4KgZUqBpC_3oOrb8G1uTEk4AOneAKH8I3Iy4yN5hXIg5wWu4-I8Oxk1ZCmw4NeHIcDoE_7AuJoCQb3weCgiidFp759R4a4QOp1yXN0Beg4OTsPFK0)

dml语句还有不少内部结构，简单分类一下:

字段相关部分,其中WildCardField是一种特殊的SelectField,FieldList包括多个SelectField

![字段](http://www.plantuml.com/plantuml/png/AqXCpavCJrMevb9GICv9B2vMS2mkpapFoqtDAr6miL4eJYrvicFjq_KxdysVy6J7ggThfpzRj_N5rkwd3NjUDgzusj6cO6S7rngUcPFYd5YKufQPcfC2qgtrVazFefxMyxMTJtSiUzamwsLhx_Croo2686iCJir9JIw1QtisV-cBzOiWodIUzhG-wru3r_oueV75mXKlzkrxkgSVo8Oe0bcmTn5G1DbGi74-cSLGVu1i07hb-QmMPED16ce1)

表相关部分，其中从外到里关系是TableRefsClause - Join - TableSource - TableName

![表](http://www.plantuml.com/plantuml/png/AqXCpavCJrMevb9GICv9B2vMyCzppizBoIp9pCzJiB5HoChFp7koO-tpMItvUIyMhdYnRz_JFVqATdPmzI69IJcfVecvgI3LG1KOSi7vfKN9CDbdKpSywrttRDU4ztjwdlQqFkjU0nHytD3uTEs4P_ENlbY_2CWkpGFQ_NnjvzEzYz3iyW8w1cJGqzRDBngdW8pedGetyTcCzH3nvvqxdwthmjGDTarGQbgnSqvYQJ7OHI1jOFrFzqnzFWN5xQ4WLmIdvgGcb_Xa5fU0L8CTzcBzsgVzIo51KWoMVjen7UhlMW00)

条件相关部分,就不解释了

![条件](http://www.plantuml.com/plantuml/png/AqXCpavCJrMevb9GICv9B2vMSAhqB4dDLR2nKSWlIaajKaYgr4yloYyj03AUJkXxENVHo-OLJplQ5Epiu5hSGV9EbSuvYQN5gI2TS0qTRSztjo0hzKWioynBHwZC0FDVh5_xj7yfiZgVpDpC4f227103QCxBXLkHvVr1RaEs4kROPtsJdkxg1ocj1G00)

## DDL语句 ddl.go

## 表达式语句 expression.go

## 函数 functions.go

## 系统管理语句 misc.go
