<template>
  <div class="uploadWrap">
    <!-- 按钮 -->
    <div class="upload">
      <input
        type="file"
        class="fileUp"
        @change="uploadChange"
        multiple
        :accept="acceptTypes"
      />
      <button>点击上传</button>
    </div>
    <!-- 提示文字 -->
    <span class="tips">
      只能上传小于{{ maxSize }}M的
      <span v-for="type in fileTypes" :key="type">
        {{ type }}
      </span>
      格式图片,自动过滤
    </span>
    <transition-group appear tag="ul">
      <!-- 上传标签 -->
      <li class="imgWrap" v-for="item in fileList" :key="item.src">
        <!-- 图片 -->
        <div class="left">
          <img :src="item.src" />
        </div>
        <!-- 右边文字和进度 -->
        <div class="right">
          <span class="name">{{ item.name }} </span>
          <!-- <span class="num">
            <span>{{ item.progress }} %</span>
            <span
              class="continue"
              v-if="item.isFail"
              @click="continueUpload(item)"
              >重试</span
            >
          </span>
          <div class="bar" :style="`width:${item.progress}%`"></div> -->
        </div>
        <!-- 取消上传标签 -->
        <span class="cancle" @click="removeImg(item)">×</span>
        <!-- 上传成功和失败tips -->
        <span
          v-if="item.status == 1 || item.status == 3"
          :class="[
            'flag',
            item.status == 3 ? 'redBd' : item.status == 1 ? 'greenBd' : ''
          ]"
        >
          <span>{{
            item.status == 3 ? "✗" : item.status == 1 ? "✓" : ""
          }}</span>
        </span>
      </li>
    </transition-group>
    <el-button @click="uploadFile">上传</el-button>
  </div>
</template>

<script>
export default {
  name: "elupload",
  data() {
    return {
      fileList: [] /* 文件列表 */,
      fileTypes: ["jpeg", "png"],
      maxSize: 100
    };
  },
  methods: {
    uploadFile() {
      this.asyncUpload(this.fileList, 0);
    },
    asyncUpload(fileList, fileLength) {
      if (fileList.length > fileLength) {
        const data = new FormData();
        data.append("file", fileList[fileLength]);
        this.$http
          .post(
            "http://192.168.10.65:8002/jsaf-cloud-bussiness-gridfs/ajaxUpload",
            data.fileSource,
            {
              headers: {
                "Content-Type": "multipart/form-data"
              }
            }
          )
          .then(res => {
            fileList[fileLength].status = 1;
          })
          .catch(res => {
            fileList[fileLength].status = 3;
            this.$message.error("第" + (fileLength+1) + "张上传失败");
            this.asyncUpload(fileList, ++fileLength);
          });
      }
    },
    uploadChange(e) {
      let fileFilterList = this.FilerFile(
        [...e.target.files],
        this.fileTypes,
        this.maxSize
      );
      fileFilterList.map(element => {
        this.fileList.push({
          name: element.name,
          type: element.type,
          src: window.URL.createObjectURL(element),
          status: 0, //'0: 为上传 1: 已上传 2. pending 3. 上传失败 '
          size: element.size,
          fileSource: element
        });
        // console.log(processedFile);
      });
      this.$emit("onChange", fileFilterList);
    },
    FilerFile(file, fileTypes, maxSize) {
      let fileList = [];
      file.forEach(element => {
        let index = element.name.lastIndexOf(".");
        let ext = element.name.slice(index + 1);
        if (!this.fileTypes.includes(ext)) {
          throw new Error("文件格式不对");
        }
        if (element.size > maxSize * 1024 * 1024) {
          throw new Error("文件太大");
        }
        fileList.push(element);
      });
      return fileList;
    },
    removeImg(item) {
      this.fileList.splice(item, 1);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scope>
/* 上传 */
.uploadWrap {
  transition: all 0.5s ease-in-out;
  -webkit-user-select: none; /*webkit浏览器*/
  -ms-user-select: none; /*IE10*/
  -khtml-user-select: none; /*早期浏览器*/
  user-select: none;
  position: relative;
  box-sizing: border-box;
  padding: 5px;
}
.fileUp {
  opacity: 0;
  position: absolute;
  width: 73px;
  height: 27px;
}
.upload {
  padding: 5px 5px 5px 0px;
  button {
    background-color: #409eff;
    border: none;
    border-radius: 2px;
    box-sizing: border-box;
    padding: 5px 10px;
    color: white;
    outline: none;
    cursor: pointer;
  }
}
/* 提示 */
.tips {
  display: inline-block;
  font-size: 10px;
  color: #6b6972;
}
/* 图片 */
.imgWrap {
  margin: 10px 0;
  width: 400px;
  box-sizing: border-box;
  padding: 10px;
  border: 1px solid #c4d0dc;
  position: relative;
  border-radius: 5px;
  display: flex;
  overflow: hidden;
  img {
    display: block;
    width: 80px;
    height: 80px;
    object-fit: cover;
    border-radius: 5px;
  }
  > div {
    font-size: 15px;
    display: flex;
    // align-items: center;
    flex-direction: column;
    justify-content: space-around;
    width: 100%;
    padding-left: 10px;
    color: #647090;
    .name {
      overflow: hidden;
      width: 80%;
      text-overflow: ellipsis;
      white-space: nowrap;
    }
    .bar {
      width: 100%;
      height: 5px;
      background-color: #409eff;
      border-radius: 5px;
    }
  }
  .cancle {
    color: #999;
    font-size: 25px;
    position: absolute;
    right: 4px;
    top: -4px;
  }
  &:hover .flag {
    display: none;
  }
}
/* 勾 */
.flag {
  position: absolute;
  width: 0px;
  height: 0px;
  border: 20px solid #13ce66;
  top: 0;
  right: 0;
  font-size: 14px;
  color: white;
  cursor: pointer;
  span {
    position: absolute;
    font-size: 20px;
    right: -16px;
    top: -22px;
  }
}
.redBd {
  border-color: orangered orangered transparent transparent;
}
.greenBd {
  border-color: #15d169 #15d169 transparent transparent;
}
ul {
  list-style: none;
  padding: 0;
}
// 淡入淡出效果
.v-enter,
.v-leave-to {
  opacity: 0;
  transform: translate(150, 180);
  transition: all 0.5s cubic-bezier(0.55, 0, 0.1, 1);
}
.v-enter-active,
.v-leave-active {
  transition: all 0.7s cubic-bezier(0.55, 0, 0.1, 1);
}
.v-move {
  transition: all 0.5s cubic-bezier(0.55, 0, 0.1, 1);
}
.v-leave-active {
  position: absolute;
}
</style>
