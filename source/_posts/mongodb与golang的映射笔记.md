title: mongodb之golang官方驱动包bson对象笔记
author: Tany
tags:
  - golang
  - mongo
categories:
  - mongo
date: 2021-06-25 17:26:00
---
学习 mongo官方包的简单笔记，记不住了就来看看。

<!-- more -->


#### bson 源码

```
// Copyright (C) MongoDB, Inc. 2017-present.
//
// Licensed under the Apache License, Version 2.0 (the "License"); you may
// not use this file except in compliance with the License. You may obtain
// a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
//
// Based on gopkg.in/mgo.v2/bson by Gustavo Niemeyer
// See THIRD-PARTY-NOTICES for original license terms.

// +build go1.9

package bson // import "go.mongodb.org/mongo-driver/bson"

import (
	"go.mongodb.org/mongo-driver/bson/primitive"
)

// Zeroer allows custom struct types to implement a report of zero
// state. All struct types that don't implement Zeroer or where IsZero
// returns false are considered to be not zero.
type Zeroer interface {
	IsZero() bool
}

// D is an ordered representation of a BSON document. This type should be used when the order of the elements matters,
// such as MongoDB command documents. If the order of the elements does not matter, an M should be used instead.
//
// Example usage:
//
// 		bson.D{{"foo", "bar"}, {"hello", "world"}, {"pi", 3.14159}}
type D = primitive.D

// E represents a BSON element for a D. It is usually used inside a D.
type E = primitive.E

// M is an unordered representation of a BSON document. This type should be used when the order of the elements does not
// matter. This type is handled as a regular map[string]interface{} when encoding and decoding. Elements will be
// serialized in an undefined, random order. If the order of the elements matters, a D should be used instead.
//
// Example usage:
//
// 		bson.M{"foo": "bar", "hello": "world", "pi": 3.14159}
type M = primitive.M

// An A is an ordered representation of a BSON array.
//
// Example usage:
//
// 		bson.A{"bar", "world", 3.14159, bson.D{{"qux", 12345}}}
type A = primitive.A

```

#### bson.D

- document

```
// D is an ordered representation of a BSON document. This type should be used when the order of the elements matters,
// such as MongoDB command documents. If the order of the elements does not matter, an M should be used instead.
//
// Example usage:
//
// 		bson.D{{"foo", "bar"}, {"hello", "world"}, {"pi", 3.14159}}
type D []E

```

表示有序bson文档，传参使用。顺序不能乱，比如筛选，分组，求和。

#### bson.E

- element

```
// E represents a BSON element for a D. It is usually used inside a D.
type E struct {
	Key   string
	Value interface{}
}
```

表示Key,Value 属性。

#### bson.M

- map

```
// M is an unordered representation of a BSON document. This type should be used when the order of the elements does not
// matter. This type is handled as a regular map[string]interface{} when encoding and decoding. Elements will be
// serialized in an undefined, random order. If the order of the elements matters, a D should be used instead.
//
// Example usage:
//
// 		bson.M{"foo": "bar", "hello": "world", "pi": 3.14159}.
type M map[string]interface{}
```

表示无序bson文档，map类型。

#### bson.A

- array

```
// An A is an ordered representation of a BSON array.
//
// Example usage:
//
// 		bson.A{"bar", "world", 3.14159, bson.D{{"qux", 12345}}}
type A []interface{}
```

表示有序bson数组。

### 同时使用 or , and 查询

- 实现类似 where k = v  and ( takeruser = a or makeruser = b )

```
    terms :=  map[string]interface{}{}
	filter := bson.D{}
	for k, v := range terms {
		if k == "userId" {
			fltr := bson.E{
				Key: "$or",
				Value: bson.A{bson.M{
					"takeruser": v,
				}, bson.M{
					"makeruser": v,
				}},
			}
			filter = append(filter, fltr)
		} else {
			filter = append(filter, bson.E{
				Key:   k,
				Value: v,
			})
		}
	}

```