---
title: z-index 和层叠上下文
date: 2021-09-29 14:32:36
categories: CSS
tags:
- css
- z-index
permalink: z-index-and-stacking-context
---
#### 一、不含z-index的堆叠规则
1.有设置了position属性的兄弟元素，不管他们的position值为何，都按照它们在HTML结构中出现的顺序堆叠

2.没有设置position属性的元素，始终在定位元素的下层，即便他们在HTML结构中位于较晚的位置
<!--more-->

[demo:不含z-index的堆叠](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Positioning/Understanding_z_index/Stacking_without_z-index)

#### 二、设置了z-index的堆叠规则
1.z-index只对指定了position属性的元素有效

2.当没有指定z-index的时候， 所有元素都在会被渲染在默认层(0层)

3.当多个元素的z-index属性相同的时候(在同一个层里面)，会按照不含z-index的规则堆叠

4.子元素的层级还取决于父元素：当一个元素的内容发生层叠后，该元素将被作为整体在父级层叠上下文中按顺序进行层叠

[demo:层叠上下文](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Positioning/Understanding_z_index/The_stacking_context)

[demo:使用 z-index](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Positioning/Understanding_z_index/Adding_z-index)
