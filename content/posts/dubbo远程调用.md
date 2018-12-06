---
title: "Dubbo服务远程调用"
author: "ZhuTail"
tags: ["Dubbo"]
categories: ["Java"]
date: 2018-11-29
---

### Dubbo服务远程调用

### 语法糖

#### 远程连接

* telnet ip 端口

  ```shell
  telnet 127.0.0.1 50888
  ```


#### 远程调用

* 查看服务

  * 所有的服务

    ```shell
    ls
    ```

  * 指定服务 

    ```shell
    ls com.zkh360.service.punchout.product.ProductService
    ```

* 调用

  * 参数为List

    ```java
    invoke com.zkh360.service.punchout.product.ProductService.getProductsNoMatterState('A19947',['AG9655','GV2605','HN7813'])
    ```

  * 参数为自己创建的对象

    ```java
    invoke core.service.user.UserService4Dr.queryCustomerAccount({"class":"core.model.support.ZkGridCondition","rows":10,"page":1,"params":{"customerCode":"A12345"}})
    invoke core.service.user.UserService4Dr.queryCustomerAccount({"class":"core.model.support.ZkGridCondition","rows":10,"page":1,"params":{"customerCode":"A12345","userName":"test"}})
    ```

    `
