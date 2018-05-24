#### vue-imageClip ####

vue图片上传预览裁剪组件，支持移动端放大缩小平移。


### 在线预览 ###
[在线预览地址](http://admin.binlive.cn/mavatar "预览地址")

###### 二维码预览
![binlive前端开发,web开发,node,vue,react,webpack](http://img.binlive.cn/upload/1525271432704qrcode.png)
### 使用 ###
安装

    $ npm install vue-imgclip
引入

    import VimageClip from 'vue-imgclip'

注册组件后创建`<VimageClip />`标签

      // script
      components: {
        VimageClip
      }

	  // html
	  <VimageClip @imageClipper="handleclip"/>


### 方法 ###


图片裁剪: @imageClipper="handleclip"
图片裁剪方法，回调中可以获取裁剪完成base64和图片原始信息。

      methods: {
        handleclip(data) {
          console.log(data);
        }
      }
### 组件参数 ###

    <VimageClip
      width="300px"
      height="300px"
      backgroundColor="#ff6633"
      :hd=false
      :control="true"
      :buttonValue='{ clipText: "裁剪", clipClass: "clip-button"}'
      @imageClipper="handelclip"
    />

|参数   |值   |描述   |
| ------------ | ------------ | ------------ |
| width  |(string)默认200px   | 不传则默认为生成200px宽的头像上传域  |
| height  |(string)默认200px   | 不传则默认为生成200px高的头像上传域  |
|  backgroundColor | (string)默认为空  | 不传则裁剪时空的区域为透明  |
|  hd |  (boolean)默认为true  |  默认为生成两倍大小图片，解决高清屏中图片生成不清晰 |
|  control |  (boolean)默认为false  |  默认不显示控制器，可在pc端中显示调整图片大小的控制器 |
|  buttonValue |  (object)默认为{clipText: '裁剪', clipClass: 'button', resetText: '重置', resetClass: 'button'}  |  生成的裁剪和重置按钮属性，clipText为裁剪按钮文字属性，resetText为重置按钮，clipClass和resetClass为两个按钮的class |
|  imageClipper |  (function)图片裁剪回调方法  |  function(data),data为图片裁剪生成的base64和图片原始信息 |
