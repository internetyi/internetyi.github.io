---
layout: page
title: 关于
menu: About
---
{% assign current_year = site.time | date: '%Y' %}

zhny
===
男 90后

## 概况

- 邮箱：zhangzhny1#gmail.com
- 主页：[http://github.zhnytech.com](http://github.zhnytech.com)
- 微博：[@internetyi](http://weibo.com/internetyi)

计算机专业毕业，{{ current_year | minus: 2012 }} 年在职工作经验，{{ current_year | minus: 2012 }} 年 web 开发经验。

## 教育
- 上海第二工业大学 — 大专 2009 - 2012

## keywords
<div class="btn-inline">
{% for keyword in site.skill_keywords %} <button class="btn btn-outline" type="button">{{ keyword }}</button> {% endfor %}
</div>

### 综合技能

| 名称 | 熟悉程度
|--:|:--|
| PHP | ★★★★★ |
| javascript | ★★★★☆ |
| Linux | ★★★★☆ |
| Nodejs | ★★★★☆ |
| Markdown | ★★★★★ |
| C | ★★★☆☆ |
| Photoshop | ★★★★☆ |
