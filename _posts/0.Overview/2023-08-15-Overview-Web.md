---
layout: post
title: "Web概要"
date: 2023-08-15
tag: Overview
---   




## 目录
* [HTML+CSS](#content1)
* [JavaScript](#content2)



<!-- ************************************************ -->
## <a id="content1">HTML+CSS</a>

#### **一、选择器**     

**1、选择器的介绍**    
标签选择器 `div { }`      
id选择器 `#content { }`         
类选择器 `.word { }`   
属性选择器 `[title] { }`   
后代选择器 `div span { }`   
子选择器(直接子元素) `div > span { }`    
兄弟选择器(相邻兄弟) `div + p { }`    
兄弟选择器(全体兄弟) `div~p { }`    
交集选择器 `div, .one { }`    
并集选择器 `div.one { }`

**2、选择器的优先级**   

id选择器 > 类选择器、属性选择器 > 标签选择器     
```css
#content {
    background-color: blue;
}

.word {
    background-color: green;
}

[title] {
    background-color: yellow;
}

div {
    color: white;
    background-color: red;
    line-height: 80px;
    padding-left: 30px;
}
```

组合权重：   
`#hehe.hello > [title="xiaohei"].hello > div.hello`    
```css
#hehe.hello {
    background-color: orange;
}

[title="xiaohei"].hello {
    background-color: green;
}

div.hello {
    background-color: red;
}
```

#### **二、伪类**     

**1、动态伪类**    
```css
/* 未访问的链接 */
a:link {
    color: red;
}

/* 已访问的链接 */
a:visited {
    color: green;
}

/* 通过按下 Tab 键 导航到链接时触发。 或者按下鼠标左键松手后触发*/
a:focus {
    color: yellow;
}

/* 当鼠标移动到链接上 */
a:hover {
    color: blue;
}

/* 被激活的链接（当鼠标左键单击链接时，未松开） */
a:active {
    color: yellow;
}
```

**2、结构伪类**    

```css
 /* span元素 并且它是父元素中的第3个子元素 */
span:nth-child(3) {
    color: red;
}

/* 4n+2的可能值是2、6、10、14、18... */
span:nth-child(4n + 2) {
    background-color: yellow;
}

/* an + b */
/* a=-1, b=2*/
/* -n + 2的可能值是2、1 */
span:nth-child(-n + 2) {
    color: red;
}

span:nth-last-child(2n) {
    color: red;
}
```



#### **三、属性的继承**    

可以继承的属性     
`line-height`   
`font-size`    
`color`    
`text-align`    

```css
.container {
    background-color: blue;
    height: 100px;

    line-height: 60px;
    font-size: 20px;
    color: white;
    text-align: center;
}

.container .sub1 {
    /* background-color: inherit; */
    background-color: green;
    /* line-height: 100px; */
}

.container .sub2 {
    background-color: red;
}
```



<!-- ************************************************ -->
## <a id="content2">JavaScript</a>








----------
>  行者常至，为者常成！



