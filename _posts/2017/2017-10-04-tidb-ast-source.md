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

## DML语句 dml.go

dml语句类型
![dml语句类型](http://www.plantuml.com/plantuml/png/Iyv9B2vMS4dDIIr93Ix9BL6miL580VDUh5_xj7-fWfqTLqfkZbz-Igg2JOskBf9IhcImNi-yujIY4fZUJ30FXrw4KgZUqBpC_3oOrb8G1uTEk4AOneAKH8I3Iy4yN5hXIg5wWu4-I8Oxk1ZCmw4NeHIcDoE_7AuJoCQb3weCgiidFp759R4a4QOp1yXN0Beg4OTsPFK0)
