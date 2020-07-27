<template>
  <div class="demo-box">
    <div class="upload-btn">
      <span>上传图片</span>
      <input type="file" class="file-btn" ref="fileBtn" @change="uploadImg" accept="image/*" />
    </div>
    <div class="move-content">
      <img
        :src="imageUrl"
        ref="changeImg"
        id="changeImg"
        @touchstart="startTouch"
        @touchmove="moveTouch"
        @touchend="endTouch"
        style="transform: translate(0px,0px) translateZ(0px)"
      />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      url: "",
      dis: [], // 记录双指移动的数据
      touchX: 0, // 开始x坐标
      touchY: 0, // 开始y坐标
      touchX1: 0, // 结束x坐标
      touchY1: 0, // 结束y坐标
      x: 0, // 手指1x坐标
      y: 0, // 手指1y坐标
      x1: 0, // 手指2x坐标
      y1: 0, // 手指2y坐标
      disMinOld: 1, // 初始放大倍数
      disMin: 1, // 实际放大倍数
      transX: 0, // 记录某时刻x偏移量
      transY: 0, // 记录某时刻y偏移量
      posX: 0, // 最终x偏移量
      posY: 0, //最终y偏移量
      matrix: "", // 记录tramsform的属性
      length: 0, // 手指长度
      showImg: false,
      scaleByPhone: 1, // 橡树比
      posterBox: null, // 生成的节点
      canvasImg: "",
      imageUrl: "http://h5.joyoung-ad.com/00images/200707jinrongjie/share.jpg",
      oldImgurl: "",
      imgInfo: null,
      angle: 0, //旋转角度
      olddis: 0,
      angleList: [],
    };
  },
  methods: {
    async uploadImg() {
      var that = this;
      let inputFile = await that.$refs.fileBtn.files[0];
      let size = inputFile.size; // 读取文件的大小
      let maxSize = 7 * 1024 * 1024; // 限定最大文件的大小
      if (size > maxSize) {
        alert("不能上传大于7M的图");
      } else {
        let res;
        if (inputFile) {
          // 使用FileReader去读取file对象
          const reader = new FileReader();
          res = reader.readAsDataURL(inputFile);
          reader.onloadend = function () {
            // 把图片转base64
            var strBase64 = reader.result.substring(0);
            that.imageUrl = strBase64;
          };
        } else {
          alert("上传失败");
        }
      }
    },
    // 触碰屏幕事件
    startTouch(e) {
      this.length = e.touches.length; // 触碰屏幕的手指数
      this.x = Number(e.touches[0].clientX); //触碰的x坐标
      this.y = Number(e.touches[0].clientY); // 触碰的Y坐标
      if (this.length == 1) {
        // 获取图的transform属性
        this.matrix = document.defaultView.getComputedStyle(
          document.getElementById("changeImg"),
          null
        ).transform;
        // 截取x偏移量
        this.transX = parseFloat(
          this.matrix.split("(")[1].split(")")[0].split(", ")[4]
        );
        // 截取y偏移量
        this.transY = parseFloat(
          this.matrix.split("(")[1].split(")")[0].split(", ")[5]
        );
      } else if (this.length == 2) {
        this.touchX = Number(e.touches[1].clientX); //触碰的x坐标
        this.touchY = Number(e.touches[1].clientY); // 触碰的Y坐标
        this.touchX1 = Number(e.touches[0].clientX); //触碰的x坐标
        this.touchY1 = Number(e.touches[0].clientY); // 触碰的Y坐标
      }
    },
    // 手指移动事件
    moveTouch(e) {
      e.preventDefault();
      try {
        // 一只手指
        if (this.length == 1) {
          this.x1 = Number(e.touches[0].clientX);
          this.y1 = Number(e.touches[0].clientY);
          this.posX = parseFloat(this.x1 - this.x);
          this.posY = parseFloat(this.y1 - this.y);
          this.posX = this.posX + this.transX;
          this.posY = this.posY + this.transY;
          document
            .getElementById("changeImg")
            .setAttribute(
              "style",
              "transform: translate(" +
                this.posX +
                "px," +
                this.posY +
                "px) scale(" +
                this.disMinOld +
                ") rotate(" +
                this.angle * 2 +
                "deg)"
            );
        } else if (this.length == 2) {
          // // 两只手指
          this.x = Number(e.touches[1].clientX);
          this.y = Number(e.touches[1].clientY);
          this.x1 = Number(e.touches[0].clientX);
          this.y1 = Number(e.touches[0].clientY);
          // sqrt 平方根
          // pow x 的 y 次幂
          // 计算两点距离
          this.dis.push(
            Math.sqrt(
              Math.pow(this.x - this.x1, 2) + Math.pow(this.y - this.y1, 2)
            )
          );

          if (this.dis.length > 1) {
            this.disMin = this.dis[this.dis.length - 1] / this.dis[0];
            this.disMin = this.disMinOld * this.disMin;
            document
              .getElementById("changeImg")
              .setAttribute(
                "style",
                "transform: translate(" +
                  this.posX +
                  "px," +
                  this.posY +
                  "px) scale(" +
                  this.disMin +
                  ") rotate(" +
                  this.angle * 2 +
                  "deg)"
              );
          }
        }
      } catch (e) {
        console.log("touchMoveFunc：" + e.message);
      }
    },
    // 手指离开屏幕
    endTouch(e) {
      this.dis = [];
      this.touchX = 0;
      this.touchX1 = 0;
      this.touchY = 0;
      this.touchY1 = 0;
      this.disMinOld = this.disMin;
    },
  },
};
</script>

<style lang="less" scoped>
.demo-box {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  .upload-btn {
    width: 10rem;
    height: 4rem;
    overflow: hidden;
    position: relative;
    background: bisque;
    border: 1px solid #f3f3f3;
    border-radius: 8px;
    line-height: 4rem;
    font-size: 14px;
    text-align: center;
    .file-btn {
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 0;
      opacity: 0;
    }
  }
  .move-content {
    width: 37.5rem;
    height: 40rem;
    margin-top: 10rem;
    background: #000;
    text-align: center;
    overflow: hidden;
    display: table-cell;
    vertical-align: middle;
    text-align: center;
    img {
      width: auto;
      height: auto;
      max-width: 100%;
      max-height: 100%;
    }
  }
  .poster-img {
    width: 50vw;
    height: auto;
  }
}
</style>
