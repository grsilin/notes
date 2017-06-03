# Markdown syntax

## 简要语法

标题：#

```html
    # 一级标题
    ## 二级标题
    ### 三级标题
    ### 四级标题
```

列表：- 或 * 或 1. 2.

```html
    <!--无序列表-->
    * 1
    * 2
    * 3

    <!--有序列表-->
    1. list
    2. list
    3. list
```

* list
* list
* list

引用：>

```html
    > 这里是引用的内容
```

> 这里是引用的内容

多层级引用：

> balabalabala
>
>> lalala
>
>balabalabala

图片: `![](){ImageCap}{/ImageCap}`

![Markdown](http://i4.buimg.com/1949/cbbc12ba42ed8fd2.jpg)

链接: `[]()`

[google](https://google.com)

粗体和斜体: `*` 和 `**`

 当我想把这里变成斜体时，这就成了*斜体的内容*

 同样我想把这里变成粗体时，这里就成了**粗体的内容**

代码框: "`","`" 或 "```Java"，多个退格具有同样的功能

        String gradle = "vip";

分割线： *** ___ ---

这里就是传说中的分割线，很神圣的呦。
***

特殊字符：

在Markdown中可以自由的书写html中的特殊字符

&copy; < >

在code 区块内，`<` 和 `&` 这等字符都会被转换为 HTML 实体

自动链接: <>

点击<https://google.com>搜索

反斜杠：转义

在需要使用 * 等字符时，需要使用 \ 进行转义
