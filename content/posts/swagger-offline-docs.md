---
title: 根据Swagger在线数据生成离线文档
description: 基于 Swagger 接口文档生成离线 PDF 与 HTML 文档。
publishDate: 2018-08-14 17:11:14
tags:
  - Swagger
  - Java
---

### 已经在[ PR22 ](https://github.com/Swagger2Markup/spring-swagger2markup-demo/pull/22)中升级 libs 到当前最新版本

## 使用方式

```bash
git clone git@github.com:Swagger2Markup/spring-swagger2markup-demo.git
```

修改根目录 `pom.xml` 中 `swaggerInput` 为Swagger在线文档地址，例如

```xml
<swaggerInput>http://192.168.1.131:8000/v2/api-docs</swaggerInput>
```

执行

```bash
mvn clean test
```

执行完成后将在 `spring-swagger2markup-demo/target/asciidoc` 文件夹下生成 **html/pdf** 文档

## 注意

可在 `spring-swagger2markup-demo/src/docs/asciidoc` 中配置生成文件结构，使用[ AsciiDoc语法 ](https://www.w3cschool.cn/solr_doc/solr_doc-d6532jwv.html)

index.adoc

```
include::{generated}/overview.adoc[]
include::manual_content1.adoc[]
include::manual_content2.adoc[]
include::{generated}/paths.adoc[]
include::{generated}/security.adoc[]
include::{generated}/definitions.adoc[]
```

manual_content1.adoc

```
== Chapter of manual content 1

This is some dummy text

=== Sub chapter

Dummy text of sub chapter
```

manual_content2.adoc

```
== Chapter of manual content 2

This is some dummy text
```
