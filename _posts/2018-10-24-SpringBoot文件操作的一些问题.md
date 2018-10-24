---
layout:     post
title:      SpringBoot文件操作的一些问题
category: blog
description: 最近又被拉去用Springboot搭网站了，把遇到的一些坑记录下来
---

## SpringBoot文件操作的一些问题

#### 1. 文件下载

* 在开发时，我实现的一个需求是根据用户信息生成相应的word文件，然后给用户下载，于是我就用了下面的代码实现了一个文件下载的功能

  ```java
  OutputStream os = null;
  InputStream is = null;
  is = new FileInputStream(templeteFile);
  CustomXWPFDocument doc = new CustomXWPFDocument(is);
  /*省略word文件操作*/
  ...
  os = response.getOutputStream();
  doc.write(os);
  os.close();
  is.close();
  ```

* 本来觉得应该没什么问题，但是却报了下面这个问题

  >  java.lang.IllegalStateException: getOutputStream() has already been called for this response

* 于是，我在网上查了很多解决方法，发现和我的实现好像没什么区别，于是就按他们的方式瞎改了好多，比如加一些try…catch…finally等等，然而发现都没什么用

* 最后，在一个博客里看到可以这个问题的原因是由于response.getWriter()造成的，于是，在controller前面加上一个@ResponseBody注解，让这个文件数据直接写入ResponseBody中，不再进一步解析就不会遇到这样的问题了，就像这样：

  ```java
  @RequestMapping("/test")
  @ResponseBody
  public void test() {
      这一段略了，都是一样的。。。
  }
  ```

#### 2. 读取静态文件

* 由于我生成word文件时是根据模板生成的，于是我想把几个模版文件放到webapp里，然后等到用的时候直接读取就可以了，这样就省去了经常修改路径的麻烦。

* 本来以为重新打包war包的时候，直接用路径访问就可以，但是感觉好像和之前的SpringMVC不太一样，并不能用路径直接访问。比如下面这样的文件结构，直接访问 `/webapp/xxx.txt` 路径无法访问

  ```
  |-src
     |-main
        |-java
        |-resources
        |-webapp
  ```

* 使用Spring提供的ResourceUtils完成文件访问

  ```java
  if (ResourceUtils.isUrl("classpath:sql/initData.sql")) {
        try {
         File file = ResourceUtils.getFile("classpath:sql/initData.sql");
            } catch (FileNotFoundException ignored) {
        } 
   }
  ```

* 但是查阅资料，发现有一些说在linux使用这种方法会出问题，等到迁移到服务器时进行实验。

---

