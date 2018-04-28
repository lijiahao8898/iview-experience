<p align="center">
    <a href="https://www.iviewui.com">
        <img width="200" src="https://file.iviewui.com/logo.svg">
    </a>
</p>

# iView 踩坑经验
[![vue](https://img.shields.io/badge/vue-2.5.13-brightgreen.svg?style=flat-square)](https://github.com/vuejs/vue)
[![iview ui](https://img.shields.io/badge/iview-2.8.0-brightgreen.svg?style=flat-square)](https://github.com/iview/iview)

Chrome浏览器
## 1. Upload组件

> Upload组件 关于手动上传：iviwe文档关于手动上传的文档，仅仅只是一个模拟的，并未真正的上传，导致留下一个坑。

业务需求：
希望在上传之前，先选择好文件，然后再统一上传，避免上传后再去删除，浪费带宽跟时间。

通过调用before-upload钩子函数，去拦截上传的文件，return false可阻止iview自动上传；将file存于一个数组readyFiles中，用于显示出来以及可去删除选择过的文件，避免上传不需要的。


上传成功后，再调用on-success钩子函数，实现手动上传全部流程。

这一块是iview文档并未贴出来的。通过查看源码，发现可通过$refs去调用post函数实现上传功能。

主要代码：
```html
// 重点在于ref,点击上传调用uploadFile函数的时候是通过ref去调用post方法达到上传文件目的的
<Upload
  ref="upload"
  multiple
  action="//192.168.0.1/upload"
  :before-upload="handleBeforeUpload"
  :on-success="handleSuccessUpload"
  >
    <Button type="ghost" icon="ios-cloud-upload-outline" style="marginLeft:100px">上传</Button>
</Upload>
```
```js
uploadFile () {
    this.loadingStatus = true;
    for (let i = 0; i < this.readyFiles.length; i++) {
        let file = this.readyFiles[i];
        this.$refs.upload.post(file);
    }
},
```

> 最后： 这个方法通过调用iview内部的post方法，是一个文件一个文件上传的。希望一次上传多个文件，还需要改造，自己通过axios等xhr库去上传，而不是采用post方法！

## 2. Page组件

> Page组件 在某些情景下的下拉（条数选择）位置不符合预期

查看Page组件样式，发现Page的下拉（条数选择）的position属性是absolute，top与left通过计算得到。
但Page组件只会在刚开始计算一次，这样子导致的情况就是，数据条数变多时，下拉（条数选择）所在位置不符合预期。

解决方案： 在Page组件增加一条style属性，设置position为relative。
（这个应该是一个小坑，但不清楚作者不加relative的缘由，不加relative，在高度变化后不去重新计算位置，导致在某些情况下就会有不符合预期的错误）。

## 3. sidebarMenu(iview-admin组件)

> 这个提了issue，所以直接给传送门吧。

[iview版本更新导致sidebarMenu使用异常](https://github.com/iview/iview-admin/issues/592)