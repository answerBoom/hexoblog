---
title: ElementUI图片上传
tags:
  - 前端
  - vue
categories:
  - 前端
mathjax: true
abbrlink: f4ecfab5
date: 2023-06-10 20:22:43
description: ElementUI图片上传
---

# ruoyi实现不需要token访问静态资源

在config/SecurityConfig配置
```java
.antMatchers(
                        HttpMethod.GET,
                        "/",
                        "/*.html",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        //profile下有很多文件，只开放头像类的文件
                        "/profile/upload/**",
                        "/profile/**",
                        "/profile/avatar/**"
```
"/xxx/**"让xxx路径下的全部文件访问不需要token

在config/ResourcesConfig中增加
```java
        /** 本地文件上传路径 */
        registry.addResourceHandler(Constants.RESOURCE_PREFIX + "/**")
                .addResourceLocations("file:" + RuoYiConfig.getProfile() + "/");
```
# 前端加入Element Ui upload组件组件
```html
<template>
  <div ref="dasd">
    <el-upload
      class="img-upload"
      ref="upload"
      action="http://localhost:8080/dss/ServiceStd/uploadimg"
      :on-preview="handlePreview"
      :on-remove="handleRemove"
      :headers="header"
      :before-remove="beforeRemove"
      :on-success="handleSuccess"
      multiple
      :limit="1"
      :on-exceed="handleExceed"
      :file-list="fileList">
      <el-button size="small" type="primary">点击上传</el-button>
      <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
    </el-upload>
  </div>
</template>

<script>
import { getToken } from '@/utils/auth';
export default {
  name: 'ImgUpload',
  data () {
    return {
      fileList: [],
      url: '',
      header: {
        Authorization: 'Bearer ' + getToken()
      }
    }
  },

  methods: {
    handleRemove (file, fileList) {
      this.$emit("clearImageUrl")
    },
    handlePreview (file) {
    },
    handleExceed (files, fileList) {
      this.$message.warning(`当前限制选择 1 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
    },
    beforeRemove (file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    },
    handleSuccess (response) {
      this.url = response
      this.$emit('onUpload')
      this.$message.warning('上传成功')
    },
    clear () {
      this.$refs.upload.clearFiles()
    },

  }
}
</script>
```
## 解决Element ui上传图片时未携带token访问不到服务器问题
在data()里面加入header属性设置子属性Authorization
```javascript
data () {
    return {
		header: {
        Authorization: 'Bearer ' + getToken()
      }
    }
```
在el-upload里设置
```html
<el-upload
:headers="header"
>
```

## 解决upload组件重新进入上传时清除上次上传
在提交时
```html
            <el-form-item label="作业标准示范" prop="OpeStdThr">
              <el-input v-model="form.OpeStd" autocomplete="off" placeholder="图片 URL" readonly></el-input>
             <img-upload @onUpload="uploadImg" ref="imgUpload" @clearImageUrl="clearImageUrl(1)"></img-upload>
            </el-form-item>
```
在提交时新增clear()方法调用子组件里clear方法

```javascript
submitForm: function () {
          this.$refs.imgUpload.clear();
        }
```
## 删除图片时同时删除URl
![点击删除图片时同时删除URl](https://img-blog.csdnimg.cn/b8cf3964cc8040908db4096979f1c562.png#pic_center)
加入@clearImageUrl绑定方法
```html
            <el-form-item label="作业标准示范" prop="OpeStdThr">
              <el-input v-model="form.OpeStd" autocomplete="off" placeholder="图片 URL" readonly></el-input>
             <img-upload @onUpload="uploadImg" ref="imgUpload" @clearImageUrl="clearImageUrl(1)"></img-upload>
            </el-form-item>
```
clearImageUrl(1)  括号内可以直接携带参数，根据参数执行哪种方法
```javascript
    clearImageUrl(type) {
      if (type == 1) {
      	//delImgUrl调用后端接口删除已上传的照片
        delImgUrl(this.form.OpeStdOne);
        this.form.OpeStdOne = '';
      }else if (type ==2) {
        delImgUrl(this.form.OpeStdTwo);
        this.form.OpeStdTwo = '';
      }else if (type == 3) {
        delImgUrl(this.form.OpeStdThr);
        this.form.OpeStdThr = '';
      }
    }
```

### delImgUrl可以采用两种方式
 1. 使用缓存，到了一定图片数量再去删除
 2. 直接去后台删除
