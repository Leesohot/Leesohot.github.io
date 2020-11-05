---
title: DataSearch-API
url: hello_url
tags:
  - 标签1
  - 标签2
categories:
  - 分类1
  - 分类2
date: 2020-05-08 20:03:00
---
# DataSearch-服务接口指南

**版本：1.0.0**

数据模糊搜索功能

## 接口用法

命名空间：**GeoBOS.ds**

### search(words, options, list, callback)

模糊搜索功能。

#### 参数

|参数|类型|描述|
|----|----|----|
|words |`Object` or `String`|搜索的内容|
|options |`Object`|搜索的控件|
|list |`Array`|搜索的范围|
|callback |`Function`|回调函数|

#### words
words参数为`Object`时可以使用逻辑查询运算符。这些运算符用于过滤数据并根据给定条件获得精确结果。下表包含逻辑查询运算符：

|名称|描述|
|----|----|
|$and    | 返回符合所有子句条件的所有文档|
|$or | 返回符合任何子句条件的所有文档        |

##### 格式样例
```js
{ $and: [ { <expression_1> }, { <expression_2> } , ... , { <expression_N> } ] }
```

操作者执行的逻辑AND或OR表达式和选择满足所有表达式中的条目的阵列上操作。$and/$or的操作者使用短路评价：即如果$and的第一表达式的值为false，或$or的第一表达式的值为true，GeoBOS.ds.search不会评价剩余的表达式

##### $and样例
```js
const words = {
  $and: [{ author: 'abc' }, { title: 'xyz' }]
}
```
##### $or样例
```js
const words = {
  $or: [{ author: 'abc' }, { author: 'def' }]
}
```

##### 同时还可以与扩展搜索进行匹配

这种高级搜索形式使您可以微调结果。

空格用作AND运算符，而单个竖线（|）字符用作OR运算符。要转义空格，请使用双引号，例如使用`="scheme language"`进行精确匹配。

|示例|类型|示例描述|
|----|----|----|
| `jscript` | 模糊匹配 | 模糊匹配jscript的项目 |
| `=scheme` | 完全符合 | 完全符合scheme的项目 |
| `'python` | 包含匹配 | 包含python的项目 |
| `!ruby` | 完全相反的匹配 | 不包含ruby的项目 |
| `^java` | 前缀匹配 | 以java开头的项目 |
| `!^earlang` | 前缀不匹配 | 不以earlang开头的项目 |
| `.js$` | 后缀匹配 | 以.js结尾的项目 |
| `!.go$` | 后缀匹配 | 不以.go结尾的项目 |


```js
const words = {
  $and: [
    { title: 'old war' }, // Fuzzy "old war"
    { color: "'blue" }, // Exact match for blue
    {
      $or: [
        { title: '^lock' }, // Starts with "lock"
        { title: '!arts' } // Does not have "arts"
      ]
    }
  ]
}
```

#### options

options作为搜索控件有如下的属性

##### 基本选项

|属性|类型|默认值|描述|
|----|----|----|
| isCaseSensitive |`boolean`|false |指示是否区分大小写|
| includeScore |`boolean`|false|分数是否应包括在结果集中。得分0表示完全匹配，而得分1表示完全不匹配|
| includeMatches |`boolean`|false|是否将匹配项包含在结果集中。true时，结果集中的每个记录都将包含匹配字符的索引。
| minMatchCharLength |`boolean`|false |指示是否区分大小写|
| isCaseSensitive |`number`|1 |仅返回长度超过该值的匹配项。例如要忽略结果中的单个字符匹配，请将其设置为2|
| shouldSort |`boolean`| true |是否按分数对结果列表进行排序|
| findAllMatches |`boolean`| false |设置为true时，即使在字符串中已经找到了完全匹配项，匹配函数也将继续搜索模式的结尾。|
| keys |`Array`| [ ] |将搜索的键列表。这支持嵌套路径，加权搜索，字符串和对象数组的搜索|

keys属性中，可以为关键字分配权重，以在搜索结果中为其赋予更高（或更低）的值。该weight值必须大于0
```js
const options = {
  includeScore: true,
  keys: [
    {
      name: 'title',
      weight: 0.3
    },
    {
      name: 'author',
      weight: 0.7
    }
  ]
}
```

##### 模糊匹配选项

|属性|类型|默认值|描述|
|----|----|----|
| location |`number`|0 |确定大约在文本中的位置是期望找到的模式|
| threshold |`number`|0.6 |匹配算法在什么时候放弃。阈值0.0要求完全匹配(字母和位置均需匹配)，阈值1.0会匹配任何内容。|
| distance |`number`|100 |确定匹配必须与模糊位置（由指定location）之间的接近程度。|
| ignoreLocation |`boolean`|false |当使用时true，搜索将忽略location和distance，因此匹配字符出现在字符串中的位置无关紧要|

##### 高级选项

|属性|类型|默认值|描述|
|----|----|----|
| useExtendedSearch | `boolean` | false |如果为true，则启用类Unix搜索命令的使用，详见扩展words参数中扩展搜索功能示例|
| getFn | `Function` | (obj: T, path: string | string[]) => string | string[ ] | 用于在提供的路径上检索对象值的函数。默认值还将搜索嵌套路径|
| sortFn | `Function` | (a, b) => number | 用于对所有结果进行排序的函数。默认值将按相关性得分递增，索引递增排序|
| ignoreFieldNorm | `boolean` | false |当时true，相关性得分的计算（用于排序）将忽略字段长度范数。|

设置ignoreFieldNorm为true唯一有意义的时间是，无论多少个术语都无关紧要，而只查询存在术语。



#### list

搜索的范围可以是一个字符串数组或是一个对象数组

可以通过使用点（.）或数组表示法提供路径来搜索嵌套值，该路径最终必须指向字符串，否则您将不会获得任何结果

##### 样例

```js
const list = [
  {
    title: "Old Man's War",
    author: {
      name: 'John Scalzi',
      tags: [
        {
          value: 'American'
        }
      ]
    }
  },
  {
    title: 'The Lock Artist',
    author: {
      name: 'Steve Hamilton',
      tags: [
        {
          value: 'English'
        }
      ]
    }
  }
]

const options = {
  includeScore: true,
  keys: ['author.tags.value']
}

GeoBOS.ds.search('engsh', options, list, function (err,res) {
  if (!err) {
    console.log(version)
  }
})
```

#### callback

回调函数

|参数|类型|描述|
|----|----|----|
|err    |`Error` |错误信息, 调用成功时为null|
|res|`Object`|搜索的结果        |

回调函数会需要根据options的配置情况（是否包含模糊匹配得分等）返回指定属性的对象，如果只需返回结果对象，可用res.item

##### 样例

```js
const options = {
  includeScore: true,
  keys: ['msg', 'from_name', 'plotName', 'description'],
  threshold: 0.7
}

// 若只返回结果数组
GeoBOS.ds.search(word, options, list, function (err, res) {
  if (!err){
    for (let i = 0; i < res.length; i++) {
      console.log(JSON.stringify(res[i].item))
    }
  }
})
```


<!-- more -->
这是阅读全文

{% img /images/404.jpg %}
