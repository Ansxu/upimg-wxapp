<template>
  <div class="relate white commonpad">
    <div class="flex-container comtitile">
      <p>{{title}}</p>
      <p class="nums">{{imgPathArr.length}}/{{imgLenght}}</p>
    </div>
    <div class="imgs">
      <div class="imgItem" v-for="(img,index) in imgPathArr" :key="index">
        <div class="delete" @click="deleteImg(index)">×</div>
        <img :src="img" alt>
      </div>
      <img :src="addImgUrl" @click="chosseImg" v-show="isShowBtnUpload" class="compics">
    </div>
  </div>
</template>
<script>
export default {
  //title--上传图片标题
  //imgLenght--上传图片的最大数量
  //addImgUrl -- 上传图片按钮的本地路径；默认不显示 addImgUrl='/static/images/camera.png'
  //返回upImgs事件---全部图片的数组
  //  [
  //      {
  //         PicUrl:'base64:img1'
  //      },{
  //         PicUrl:'base64:img2'
  //      }
  // ]
  props: {
    title: {
      type: String,
      default: "上传图片"
    },
    // 上传图片的最大数量
    imgLenght: {
      type: Number,
      default: 4
    },
    addImgUrl: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      imgBase: [],
      imgPathArr: [],
      isShowBtnUpload: true //显示上传按钮的状态
    };
  },
  onLoad() {
    this.imgBase = [];
    this.imgPathArr = [];
    this.isShowBtnUpload = true;
  },
  methods: {
    // 选择图片
    chosseImg() {
      const that = this;
      let num = 0;
      if (that.imgPathArr.length < this.imgLenght) {
        num = this.imgLenght - that.imgPathArr.length;
        wx.chooseImage({
          count: num, //最大图片数量=最大数量-临时路径的数量
          sizeType: ["compressed"], //图片尺寸 original--原图；compressed--压缩图
          sourceType: ["album", "camera"], //选择图片的位置 album--相册选择, 'camera--使用相机
          success: res => {
            const imgPathArr = this.imgPathArr;
            this.imgPathArr = [];
            this.imgPathArr = imgPathArr.concat(res.tempFilePaths);
            console.log(res.tempFilePaths, "base");
            that.updateImg();
          }
        });
      }
    },
    // 根据临时路径，获取图片base64码。
    updateImg() {
      // 判断是否大于图片最大数量
      if (this.imgPathArr.length === this.imgLenght * 1) {
        this.isShowBtnUpload = false;
      } else {
        this.isShowBtnUpload = true;
      }
      this.imgBase = [];
      // 根据临时路径数组imgPathArr获取base64图片
      for (let i = 0; i < this.imgPathArr.length; i++) {
        wx.getFileSystemManager().readFile({
          filePath: this.imgPathArr[i], //选择图片返回的相对路径
          encoding: "base64", //编码格式
          success: res => {
            //成功的回调
            this.imgBase.push({
              PicUrl: "data:image/png;base64," + res.data.toString()
            });
            // 最后一张图片时，返回数据
            if(this.imgBase.length===this.imgPathArr.length){
            this.$emit("upImgs", this.imgBase);
            }
          }
        });
      }
      //以下两行注释的是同步方法 转化后的base路径
      // that.imgBase = wx.getFileSystemManager().readFileSync(res.tempFilePaths[0], "base64");
      // console.log("that.imgBase" + that.imgBase);
    },
    // 删除图片
    deleteImg(i) {
      this.imgBase.splice(i, 1);
      this.imgPathArr.splice(i, 1);
      if (this.imgPathArr.length < this.imgLenght * 1) {
        this.isShowBtnUpload = null;
        this.isShowBtnUpload = true;
      }
      this.updateImg();
      // console.log('点击了删除',i)
      // const imgBase = this.imgBase
      // imgBase.splice(i, 1);
      // this.imgBase = null;
      // this.imgBase = imgBase;
      // const imgPathArr = this.imgPathArr
      // imgPathArr.splice(i, 1);
      // this.imgPathArr = null;
      // this.imgPathArr = imgPathArr;
      // if (this.imgPathArr.length < this.imgLenght*1) {
      //   this.isShowBtnUpload = null;
      //   this.isShowBtnUpload = true;
      // }
      // console.log('删除成功',i,this.isShowBtnUpload,this.imgPathArr)
    }
  }
};
</script>
<style lang="scss" scoped>
.nums {
  font-size: 24rpx;
  color: #999;
  text-align: right;
}
.compics {
  width: 156rpx;
  height: 156rpx;
}
.comtitile {
  margin-bottom: 20rpx;
}
.comtitile p:first-child {
  font-weight: bold;
}
.imgs {
  display: flex;
  align-items: center;
  flex-flow: row wrap;
  margin-right: -20rpx;
  width: 710rpx;
  overflow: hidden;
  padding-top: 20rpx;
  .imgItem {
    position: relative;
  }
  img {
    width: 156rpx;
    height: 156rpx;
    margin-right: 20rpx;
    margin-bottom: 20rpx;
  }
  .delete {
    width: 45rpx;
    height: 45rpx;
    line-height: 43rpx;
    background: rgba(0, 0, 0, 0.6);
    color: #fff;
    font-size: 38rpx;
    text-align: center;
    border-radius: 50%;
    position: absolute;
    top: -15rpx;
    right: 5rpx;
  }
}
</style>


