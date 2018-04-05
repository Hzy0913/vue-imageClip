<template>
  <div class="image-Clip">
    <div class="Clip-wpapper" :style="{width: width, height: height}">
      <input type="file" :style="{width: width, height: height, opacity: 0}" @change="uploadImages" class="Clip-input" v-if="showFile">
      <div class="image-Clip-canvasWrapper" ref="dragImageWrapper" :style="{width: width, height: height, display: showCanvasWrapper}">
        <img src="" class="drag-image" ref="dragImage">
      </div>
      <img src="" ref="render" class="image-render" :style="{width: width, height: height, display: showRender}">
    </div>
    <button v-on:click="imageClipper" :class="buttonValue.clipClass || 'button'">{{buttonValue.clipText || '裁剪'}}</button>
    <button @click="resetImage" :class="buttonValue.resetClass || 'button'">{{buttonValue.resetText || ' 重置'}}</button>
  </div>
</template>
<script>
  import Hammer from 'hammerjs'
  import html2canvas from 'html2canvas'
  import EXIF from 'exif-js'
  export default {
    name: 'VimageClip',
    data () {
      return {
        transform: null,
        showCanvasWrapper: 'none',
        showRender: 'none',
        showFile:true,
        file: null,
        imgWidth: null,
        imgHeight: null
      }
    },
    props: {
      width: {
        type: String,
        default: '200px'
      },
      height: {
        type: String,
        default: '200px'
      },
      hd: {
        type: Boolean,
        default: true
      },
      backgroundColor: {
        type: String,
        default: ''
      },
      buttonValue: {
        type: Object,
        default: {
          clipText: '裁剪',
          clipClass: "",
          resetText: '重置',
          resetClass: "",
        }
      },
      restValue: {
        type: Number,
        default: 20
      }
    },
    created() {
      this.$root.eventHub.$emit('hidebar', false)
    },
    mounted() {
      var reqAnimationFrame = (function () {
        return window[Hammer.prefixed(window, 'requestAnimationFrame')] || function (callback) {
          window.setTimeout(callback, 1000 / 60);
        };
      })();
      var self = this;
      var dragEl = this.$refs.dragImage;
      this.START_X = 0;
      this.START_Y = 0;
      var initScale = 1;
      var ticking = false;
      this.transform;
      var mc = new Hammer.Manager(dragEl);
      mc.add(new Hammer.Pan({ threshold: 0, pointers: 0 }));
      mc.add(new Hammer.Swipe()).recognizeWith(mc.get('pan'));
      mc.add(new Hammer.Rotate({ threshold: 0 })).recognizeWith(mc.get('pan'));
      mc.add(new Hammer.Pinch({ threshold: 0 })).recognizeWith([mc.get('pan'), mc.get('rotate')]);
      mc.on("panmove", onPan);
      mc.on("pinchmove", onPinch);
      mc.on("panend", function(e) {
        self.START_X = self.transform.translate.x;
        self.START_Y = self.transform.translate.y;
      });
      mc.on("pinchend", function(e) {
        initScale = self.transform.scale;
      });
      function resetElement() {
        self.transform = {
          translate: { x: self.START_X, y: self.START_Y },
          scale: 1,
          angle: 0,
          rx: 0,
          ry: 0,
          rz: 0
        };
        requestElementUpdate();
      };
      function updateElementTransform() {
        var value = [
          'translate3d(' + self.transform.translate.x + 'px, ' + self.transform.translate.y + 'px, 0)',
          'scale(' + self.transform.scale + ', ' + self.transform.scale + ')',
          'rotate3d('+ self.transform.rx +','+ self.transform.ry +','+ self.transform.rz +','+  self.transform.angle + 'deg)'
        ];
        value = value.join(" ");
        dragEl.style.webkitTransform = value;
        dragEl.style.mozTransform = value;
        dragEl.style.transform = value;
        ticking = false;
      };
      function requestElementUpdate() {
        if(!ticking) {
          reqAnimationFrame(updateElementTransform);
          ticking = true;
        }
      };
      function onPan(ev) {
        self.transform.translate = {
          x: self.START_X + ev.deltaX,
          y: self.START_Y + ev.deltaY
        };
        requestElementUpdate();
      };
      function onPinch(ev) {
        if (ev.scale > 1) {
          self.transform.scale = initScale + ((ev.scale - 1)/2)
        } else {
          self.transform.scale = initScale - ((1 - ev.scale)/2)
        }
        requestElementUpdate();
      }
      resetElement();
    },
    methods: {
      uploadImages(files) {
        this.showCanvasWrapper = 'block';
        this.showFile = false;
        var self = this;
        var _files = files.target.files;
        var _index = 0;
        var reader = new FileReader();
        this.file = files.target.files[_index];
        reader.readAsDataURL(_files[_index]);
        reader.onload = function(event) {
          var image = new Image();
          image.src = this.result;
          image.style.opacity = 0;
          document.body.appendChild(image);
          image.onload = function() {
            self.imgWidth = image.offsetWidth;
            self.imgHeight = image.offsetHeight;
            EXIF.getData(image, function() {
              EXIF.getAllTags(this);
              var orientation = EXIF.getTag(this, "Orientation");
              var rotateCanvas = document.createElement("canvas"),
                rotateCtx = rotateCanvas.getContext("2d");
              switch (orientation) {
                case 1 :
                  rotateCanvas.width = image.offsetWidth;
                  rotateCanvas.height = image.offsetHeight;
                  rotateCtx.drawImage(image, 0, 0, image.offsetWidth, image.offsetHeight);
                  break;
                case 6 : // 顺时针 90 度
                  rotateCanvas.width = image.offsetHeight;
                  rotateCanvas.height = image.offsetWidth;
                  rotateCtx.translate(0, 0);
                  rotateCtx.rotate(90 * Math.PI / 180);
                  rotateCtx.drawImage(image, 0, -image.offsetHeight, image.offsetWidth, image.offsetHeight);
                  break;
                case 8 :
                  rotateCanvas.width = image.height;
                  rotateCanvas.height = image.width;
                  rotateCtx.translate(0, 0);
                  rotateCtx.rotate(-90 * Math.PI / 180);
                  rotateCtx.drawImage(image, -image.width, 0, image.width, image.height);
                  break;
                case 3 : // 180 度
                  rotateCanvas.width = image.width;
                  rotateCanvas.height = image.height;
                  rotateCtx.translate(0, 0);
                  rotateCtx.rotate(Math.PI);
                  rotateCtx.drawImage(image, -image.width, -image.height, image.width, image.height);
                  break;
                default :
                  rotateCanvas.width = image.width;
                  rotateCanvas.height = image.height;
                  rotateCtx.drawImage(image, 0, 0, image.width, image.height);
              }
              var rotateBase64 = rotateCanvas.toDataURL("image/png", 0.5);
              self.$refs.dragImage.src = rotateBase64;
              if (image.offsetHeight > image.offsetWidth) {
                var width = self.height.replace(/[^0-9]/ig,"");
                self.$refs.dragImage.style.height = self.height;
                self.$refs.dragImage.style.width = 'auto';
                var translatex = (width - (image.offsetWidth/(image.offsetHeight/width)))/2;
                self.START_X = translatex;
                self.$refs.dragImage.style.transform = 'translate('+translatex+'px,0px)';
              } else {
                var height = self.height.replace(/[^0-9]/ig,"");
                var translatey = (height-(image.offsetHeight/(image.offsetWidth/height)))/2;
                self.$refs.dragImage.style.width = self.width;
                self.$refs.dragImage.style.height = 'auto';
                self.$refs.dragImage.style.transform = 'translate(0px,'+translatey+'px)';
                self.START_Y = translatey;
              }
              document.body.removeChild(image);
              self.showCanvasWrapper = 'block';
            });
          }
        }
      },
      imageClipper(getDataUrl) {
        var self = this;
        if (self.showFile) return;
        var wrapperDom = this.$refs.dragImageWrapper;
        var width = this.width.replace(/[^0-9]/ig,"");
        var height = this.height.replace(/[^0-9]/ig,"");
        var scaleBy = this.hd ? 2 : 1;
        var canvas = document.createElement("canvas");
        // 获取元素相对于视窗的偏移量
        var rect = wrapperDom.getBoundingClientRect();
        canvas.width = width * scaleBy;
        canvas.height = height * scaleBy;
        canvas.style.width = width + "px";
        canvas.style.height = height + "px";
        var context = canvas.getContext("2d");
        context.scale(scaleBy, scaleBy);
        // 设置context位置, 值为相对于视窗的偏移量的负值, 实现图片复位
        context.translate(-rect.left, -rect.top);
        wrapperDom.style.backgroundColor = this.backgroundColor ? this.backgroundColor : '';
        html2canvas(wrapperDom, {
          canvas:canvas,
          logging: false,
          backgroundColor: null,
          scale:scaleBy,
        }).then(function(canvas) {
          var dataUrl = canvas.toDataURL("image/png");
          var MavatarRender = self.$refs.render;
          self.showRender = 'block';
          MavatarRender.src = dataUrl;
          const file = self.file;
          file.width = self.imgWidth;
          file.height = self.imgHeight;
          if (self) {
            self.$emit('imageClipper', {dataUrl, file});
          }
        });
      },
      resetImage() {
        var self = this;
        this.START_X = 0;
        this.START_Y = 0;
        this.transform = {
          translate: { x: 0, y: 0 },
          scale: 1,
          angle: 0,
          rx: 0,
          ry: 0,
          rz: 0
        };
        var MavatarRender = self.$refs.render;
        this.showRender = 'none';
        MavatarRender.src = ''
        var wrapperDom = this.$refs.dragImageWrapper;
        this.$refs.dragImage.src = '';;
        wrapperDom.style.backgroundColor = '';
        this.showCanvasWrapper = 'none';
        this.showFile = true;
      },
    }
  }
</script>

<style scoped>
  .Clip-input{position: absolute;left: 0px;top: 0px}
  .Clip-wpapper{background-color: #eee;position: relative;overflow: hidden; margin-bottom: 10px}
  .image-Clip-canvasWrapper{position: absolute;left: 0px;top: 0px}
  .image-Clip{display: inline-block}
  .image-render{position: absolute;left: 0px;top: 0px}
  .button{padding: 6px 12px;border-radius: 4px;outline: none;letter-spacing: 1px; border: none; border: 1px solid #eee;background-color: #fff}
</style>
