# upimg-wxapp
基于mpvue开发的微信小程序，上传图片组件

#### 先说下思路：
1. 上传图片按钮触发事件，调用[wx.chooseImage](https://developers.weixin.qq.com/miniprogram/dev/api/wx.chooseImage.html?search-key=chooseImage)方法，返回本地的临时路径
2. 遍历临时路径，读取文件管理器[wx.getFileSystemManager](https://developers.weixin.qq.com/miniprogram/dev/api/wx.getFileSystemManager.html)的本地文件内容[.readFile](https://developers.weixin.qq.com/miniprogram/dev/api/FileSystemManager.readFile.html)，根据临时地址获取base64转码

#### 贴上主要代码，封装的是一个mpvue的项目上传图片组件。
> 值得注意的是：步骤2储存的base64地址如果过长可能会出现传参出现问题，所以还是推荐在父组件执行步骤2,获取base64码，或者没选择一张图片上传一次到后台。反正代码和逻辑都在这，怎么修改都行啦

1.选择图片，保存临时地址
```
// 选择图片
    chosseImg() {
      const that = this;
      let num = 8; //上传的图片最大数量
        wx.chooseImage({
          count: num - that.imgPathArr.length, //最大图片数量=最大数量-临时路径的数量
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
    },
```
2.根据临时路径，获取图片base64码。
```
//根据临时路径，获取图片base64码。
    updateImg() {
      const that = this;
      this.imgBase = [];
      // 根据临时路径数组imgPathArr获取base64图片
      this.imgPathArr.map(item=>{
        //异步方法
        wx.getFileSystemManager().readFile({
          filePath: item, //选择图片返回的相对路径
          encoding: "base64", //编码格式
          success: res => {
            //成功的回调
            that.imgBase.push({
              PicUrl: "data:image/png;base64," + res.data.toString()
            });
            // 最后一张图片时，返回数据
            if(this.imgBase.length===this.imgPathArr.length){
            this.$emit("upImgs", this.imgBase);
            }
          }
        });
        
        //同步方法
        //let imgItem = wx.getFileSystemManager().readFileSync(item, "base64");
        //that.imgBase.push({
        //    PicUrl: "data:image/png;base64," + imgItem.toString()
        //});
        // 最后一张图片时，返回数据
        //if(this.imgBase.length===this.imgPathArr.length){
        //this.$emit("upImgs", this.imgBase);
        //}
        
      })
    }
```
最后放上封装的组件地址 https://github.com/Ansxu/upimg-wxapp.git