---
title: "前端入坑之路（一）Web Uploader实现图片的预览上传"
author: "ZhuTail"
tags: ["Web Uploader"]
categories: ["front-end"]
date: 2019-10-21
---

# Web Uploader实现图片的预览上传

> 前端之难，难于上青

## Web Uploader介绍

> WebUploader是由Baidu WebFE(FEX)团队开发的一个简单的以HTML5为主，FLASH为辅的现代文件上传组件。在现代的浏览器里面能充分发挥HTML5的优势，同时又不摒弃主流IE浏览器，沿用原来的FLASH运行时，兼容IE6+，iOS 6+, android 4+。两套运行时，同样的调用方式，可供用户任意选用。采用大文件分片并发上传，极大的提高了文件上传效率。[web uploader](http://fex.baidu.com/webuploader/)

## Web Uploader使用

### 引入资源

* 使用Web Uploader插件必须引入三种资源：JS, CSS, SWF

* ```
  <!--引入CSS-->
  <link rel="stylesheet" type="text/css" href="webuploader文件夹/webuploader.css">
  
  <!--引入JS-->
  <script type="text/javascript" src="webuploader文件夹/webuploader.js"></script>
  
  <!--SWF在初始化的时候指定-->
  ```

### HTML部分（以图片上传为例）

* ```
  <div>
      <div id="filePicker" style="margin-left: 20px">选择图片</div>
      <button id="submit" class="btn btn-blue" style="display: none">上传				</button>
  	<!--用来存放图片-->
  	<div id="fileList" class="uploader-list clearfix"></div>
  </div>
  ```

### JS部分

* ```
  /**
   * 初始化Web Uploader
   *
   * @param pickId 上传组件Id
   * @param listId 图片存储组件Id
   * @param btnId 提交按钮Id
   * @param status 1：环境照片,2：地坪/接电照片
   */
  initWebUploader: function (pickId, listId, btnId, status) {
      var uploader = WebUploader.create(
          {
              swf: 'Uploader.swf',
              server: '/scam/engineOrderManage/upload',
              pick: "#" + pickId,
              auto: false,//是否自动上传
              duplicate:true,
              fileNumLimit: 8,
              // fileSingleSizeLimit:1024*1024*2,
              accept: {
                  title: 'Images',
                  extensions: 'jpg,jpeg,png',
                  mimeTypes: 'image/*'
              }
              // ,
              // duplicate: false,
              // resize: false
          }
      );
      // 赋值给全局变量
      model.uploader = uploader;
      model.uploader.on('uploadBeforeSend', function (object, data, headers) {
          data.id = 1;
          // data.orderId = model.logActionBean.engineOrderId.split("-")[0];
          data.orderId = model.logActionBean.engineOrderId;
          data.orderStatus = status;
          data.orderType = 6;
          data.photoType = 100;
      });
  
      var $list = $("#" + listId);
      model.uploader.on('fileQueued', function (file) {
          var init = model.uploader.getFiles('inited').length;
          // 待上传的图片超过限制隐藏按钮
          if (init > 7) {
              $('#' + pickId).attr("style", "display:none;")
          }
  
          if (init > 0) {
              $('#' + btnId).attr("style", "display:block;width:86px;margin-left:20px;margin-top:5px;")
          }
  
          // 图片名称设置，看了后台代码，对文件名有要求
          file.name = "100-1-#-pic.jpg";
          var $li = $(
              '<li id="' + file.id + '" class="file-item thumbnail" style="display: flex;">' +
              '<img>' +
              // '<p class="title">' + file.name + '</p>' +
              '</li>'
          ), $img = $li.find('img');
  
          var $btns = $('<div class="file-panel">' +
                        '<span style="margin-left: 5px;">x</span></div>').appendTo($li);
          $li.on('mouseenter', function () {
              $btns.stop().animate({height: 30});
          });
          $li.on('mouseleave', function () {
              $btns.stop().animate({height: 0});
          });
  
          $img.on('click', function () {
              var _this = $(this);//将当前的img元素作为_this传入函数
              imgShow("#outerdiv", "#innerdiv", "#bigimg", _this);
          });
  
          // $list为容器jQuery实例
          $list.append($li);
  
          $btns.on('click', 'span', function () {
              var index = $(this).index();
              if (index === 0) {
                  model.uploader.removeFile(file);
                  removeFile(file);
                  return;
              }
          });
          // 创建缩略图
          // 如果为非图片文件，可以不用调用此方法。
          // thumbnailWidth x thumbnailHeight 为 100 x 100
          model.uploader.makeThumb(file, function (error, src) {
              if (error) {
                  $img.replaceWith('<span>不能预览</span>');
                  return;
              }
              $img.attr('src', src);
          }, 100, 100);
      });
  
      model.uploader.on('uploadSuccess', function (file, response) {
          model.uploader.removeFile(file);
          model.showTipDirect('tip', '图片上传成功');
      });
      // 所有文件上传成功后调用
      model.uploader.on('uploadFinished', function () {
          //清空队列
          model.uploader.reset();
      });
      model.uploader.on('error', function (type) {
          if (type == 'F_EXCEED_SIZE') {
              model.showTipDirect('tip', '文件上传超大小');
          }
          if (type == 'Q_TYPE_DENIED') {
              model.showTipDirect('tip', '错误的文件类型');
          }
          if (type == 'F_DUPLICATE') {
              model.showTipDirect('tip', "上传文件重复");
          }
      })
  
      /**
       * 删除文件
       */
      function removeFile(file) {
          var $li = $('#' + file.id);
          $li.off().find('.file-panel').off().end().remove();
          // 初始化的文件数量
          var init = model.uploader.getFiles('inited').length;
          // 待上传的图片小于过限制显示按钮
          if (init < 9) {
              $('#' + pickId).attr("style", "display:block;margin-left:20px")
          }
          if (init < 1) {
              $('#' + btnId).attr("style", "display:none;")
          }
      }
      //auto为false时,调用这个代码
      $("#" + btnId).on('click', function () {
          model.uploader.upload();
      });
  
      /**
       * 显示大图
       *
       * @param outerdiv
       * @param innerdiv
       * @param bigimg
       * @param _this
       */
      function imgShow(outerdiv, innerdiv, bigimg, _this){
          var src = _this.attr("src");//获取当前点击的pimg元素中的src属性
          $(bigimg).attr("src", src);//设置#bigimg元素的src属性
          /*获取当前点击图片的真实大小，并显示弹出层及大图*/
          $("<img/>").attr("src", src).load(function(){
              var scale = 0.8;//缩放尺寸，当图片真实宽度和高度大于窗口宽度和高度时进行缩放
              var windowW = $(window).width();//获取当前窗口宽度
              var windowH = $(window).height();//获取当前窗口高度
              var realWidth = _this.width();//获取图片真实宽度
              var realHeight = _this.height();//获取图片真实高度
              var imgWidth, imgHeight;
  
              if(realHeight>windowH*scale) {//判断图片高度
                  imgHeight = windowH*scale;//如大于窗口高度，图片高度进行缩放
                  imgWidth = imgHeight/realHeight*realWidth;//等比例缩放宽度
                  if(imgWidth>windowW*scale) {//如宽度扔大于窗口宽度
                      imgWidth = windowW*scale;//再对宽度进行缩放
                  }
              } else if(realWidth>windowW*scale) {//如图片高度合适，判断图片宽度
                  imgWidth = windowW*scale;//如大于窗口宽度，图片宽度进行缩放
                  imgHeight = imgWidth/realWidth*realHeight;//等比例缩放高度
              } else {//如果图片真实高度和宽度都符合要求，高宽不变
                  imgWidth = realWidth;
                  imgHeight = realHeight;
              }
              $(bigimg).css("width",imgWidth);//以最终的宽度对图片缩放
  
              var w = (windowW-imgWidth)/2;//计算图片与窗口左边距
              var h = (windowH-imgHeight)/2;//计算图片与窗口上边距
              $(innerdiv).css({"top":h, "left":w});//设置#innerdiv的top和left属性
              $(outerdiv).fadeIn("fast");//淡入显示#outerdiv及.pimg
          });
  
          $(outerdiv).click(function(){//再次点击淡出消失弹出层
              $(this).fadeOut("fast");
          });
  
      }},
  /**
   * 弹框关闭时重置uploader
   */
  resetUploader: function (){
      var $list = $("#fileList");
      var $list_ = $("#fileList_");
      $list.empty();
      $list_.empty();
      
      $('#submit').attr("style", "display:none;");
      $('#submit_').attr("style", "display:none;");
      // 销毁uploader，否则再次打开时会重新初始化，导致样式变大
      model.uploader.destroy();
  },
  ```

### JAVA部分

* ```
  /**
   * 工程订单图片上传
   *
   * @param files
   * @param fcScamOrderInspectionPhotoDto
   * @throws IOException
   */
  @RequestMapping(value = "/upload", method = {RequestMethod.GET, RequestMethod.POST})
  @ResponseBody
  public void upload(@RequestParam("file") MultipartFile[] files, FcScamOrderInspectionPhotoDto fcScamOrderInspectionPhotoDto) throws IOException {
  	LOG.info("-------------------开始调用上传文件upload接口-------------------");
      try {
     		fcScamOrderInspectionPhotoService.batchUploadPhotos
      	(fcScamOrderInspectionPhotoDto, files);
      } catch (Exception e) {
          e.printStackTrace();
      }
      LOG.info("-------------------结束调用上传文件upload接口-------------------");
  }
  ```

### 遇到的坑

* Web Uploader多次加载，导致选择图片的按钮样式**变大**，如果有时间重复打开关闭弹框，这个按钮能大到让你怀疑人生，直接把一无所知的我整蒙了。通过求助前端大神张导才知道，这是重复初始化导致的，并且亮了一段防止重复初始化的代码让我临摹了一波，果然解决了，在这里给助人为乐的张导点赞。`防止重复初始化的思路是设置一个全局标志位，初始化前先判断这个标志位就可以了。`
* 选择完图片之后直接关闭弹窗，然后再次打开弹窗进行图片上传，会显示多张预览图。经过分析和查找资料，发现应该是关闭弹窗的时候，没有删除队列里的图片导致的。于是尝试在 `resetUploader` 中调用`model.uploader.removeFile(file)`,但是没有起作用，通过查找api文档，找到了 `uploader.reset();//重置uploader。目前只重置了队列` 和 `uploader.destory();//销毁 webuploader 实例`，经测试 ``uploader.destory()`有效，而且同时解决了上面重复初始化的问题。
* 上传的默认参数里面有个id，是Web Uploader生成的一个字符串，如果你后端接收额外参数的对象里恰好有个id，而且不是字符串类型，就会出现错误。
* 设置上传的额外参数可以在options中设置 `uploader.options.formData = obj`，也可以在 `uploadBeforeSend`事件中处理。

## 尾巴

* 怨天尤人，不如改变自己。虽然不知道未来如何，但是技多毕竟不压身。