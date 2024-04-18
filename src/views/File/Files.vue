<template>
  <div class="app-container" style="height: 100%;overflow: hidden;">
    <el-page-header class="pageHeader" :title="'后退'" :content="'当前文件夹：' + currentLocationName" @back="goBack">
    </el-page-header>
    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5" style="float: right;">
        <el-button plain icon="el-icon-refresh" size="mini" @click="refreshGetList">刷新</el-button>
      </el-col>
      <el-col :span="1.5" style="float: right;">
        <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="addFolder">新建文件夹</el-button>
      </el-col>
      <el-col :span="1.5" style="float: right;">
        <el-button type="success" plain icon="el-icon-upload" size="mini" @click="addFile">上传文件</el-button>
      </el-col>

    </el-row>

    <!-- 文件浏览区 -->
    <div style="overflow: hidden;height: 100%;">
      <div class="drawing_card" v-loading="cardLoading">
        <template v-if="folderList.length === 0 && filesList.length === 0">
          <el-empty description="文件夹为空" style="height: 100%;width: 100%;"></el-empty>
        </template>
        <!-- 文件夹 -->
        <div class="folderContainer" v-for="( item, index ) in  folderList ">
          <div class="folderWrapper" @dblclick="doubleClickFolder(index, item)"
            @contextmenu.prevent.stop="rightClickFolder(index, item, $event)">
            <img src="@/assets/folder.png" style="width: 100px;height: 90px;"
              @contextmenu.prevent.stop="rightClickFolder(index, item, $event)" />
            <div class="folderName">
              <span>{{ item }}
              </span>
            </div>

          </div>
        </div>
        <!-- 文件 -->
        <div class="folderContainer" v-for="( item, index ) in  filesList ">
          <div class="folderWrapper" @dblclick="down(item)">
            <img src="@/assets/file.png" style="width: 100px;height: 90px;"
              @contextmenu.prevent.stop="rightClickfile(index, item, $event)" />
            <div class="folderName">
              <span>{{ item }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>


    <!-- 文件夹【右键菜单】 -->
    <div class="add-folder-9" :style="folderStyle" v-show="folderShow">
      <div class="add-folder-1">
        <div class="add-folder-2" @click="openFolder">
          打开文件夹
        </div>
        <div style="border: 2px solid rgba(18,17,42,.07)"></div>
        <div class="add-folder-2" @click="moveFolder">
          移动
        </div>
        <div style="border: 2px solid rgba(18,17,42,.07)"></div>
        <div class="add-folder-2" @click="updateFloder">
          重命名
        </div>
        <div style="border: 2px solid rgba(18,17,42,.07)"></div>
        <div class="add-folder-6" @click="deleteFolder">
          删 除
        </div>
      </div>
    </div>
    <!-- 文件【右键菜单】 -->
    <div class="add-folder-9" :style="fileStyle" v-show="fileShow">
      <div class="add-folder-1">
        <div class="add-folder-2">
          <div @click="previewfile">预览</div>
        </div>
        <div class="add-folder-2">
          <a :href="clickFilePath" download="1">下载文件</a>
        </div>
        <div style="border: 2px solid rgba(18,17,42,.07)"></div>
        <!-- <div class="add-folder-2" @click="updateFloder">
          重命名
        </div>
         <div style="border: 2px solid rgba(18,17,42,.07)"></div> -->
        <div class="add-folder-2" @click="moveFolder">
          移动
        </div>
        <div style="border: 2px solid rgba(18,17,42,.07)"></div>
        <div class="add-folder-6" @click="deleteFileFun">
          删 除
        </div>
      </div>
    </div>
    <!-- 上传文件 弹窗 -->
    <addFolder ref="addFolder1" :locationUrl="locationUrl.join('/')" />
    <moveFolder ref="moveFolder1" :moveData="moveData"></moveFolder>

    <!-- 预览 -->
    <div>
      <iframe class="iframeBox" v-if="show" :src="googleViewerUrl(prefileUrl)" width="600" height="500"></iframe>
      <i class="fa fa-times" aria-hidden="true"></i>
    </div>
  </div>
</template>
<script>

import addFolder from './components/AddFolder.vue'
import moveFolder from './components/MoveFolder.vue'
import { ListFiles, CreateFolder, DeleteF, DownloadFile } from '../../api/api'


export default {
  components: { addFolder, moveFolder },
  data() {
    return {
      historyFolderName: '',//历史文件夹name，用于【返回上一级】
      currentLocationName: 'edudoc',//当前所处位置（文件夹）名
      locationUrl: [],
      prefileUrl: "",
      //移动文件（夹）时需要的参数
      moveData: {
        typeofFolder: 0,//所选对象的类型（1：文件夹；2：文件）
        clickFolderId: -1,//被右键的文件夹id
      },
      cardLoading: false,
      folderList: [],//文件夹列表
      filesList: [],//文件列表
      // 文件夹 右键菜单栏
      folderStyle: {
        left: '0px',
        top: '0px'
      },
      folderShow: false,
      clickFolderId: -1,//被右键的文件夹id
      clickFolderName: '',//被右键的文件夹名
      // 文件 右键菜单栏
      fileStyle: {
        left: '0px',
        top: '0px'
      },
      fileShow: false,
      clickFileId: -1,//被右键的文件id
      clickFileName: '',//被右键的文件名
      clickFilePath: '',//被右键的文件路径
      show: false,
      baseUrl: "https://romantic-chipmunk-evident.ngrok-free.app/",
      localBaseUrl: "http://68575e88.r18.cpolar.top/edudoc/",
      queryParams: "edudoc"
    }
  },
  methods: {
    //返回上一级
    goBack() {
      if (this.currentLocationName == "edudoc") {
        this.$message({
          message: '这是根目录！',
          type: 'warning'
        });
      } else {
        this.queryParams = this.historyFolderName;
        this.historyFolderName = this.currentLocationName;
        this.locationUrl.pop()
        this.currentLocationName = this.locationUrl.length === 0 ? "edudoc" : this.locationUrl[this.locationUrl.length - 1]
        this.getList();
      }

    },
    // 获取列表数据
    getList() {
      this.loading = true
      let para = {
        folderPath: this.currentLocationName === "edudoc" || this.locationUrl.length === 0 ? "edudoc" : this.locationUrl.join('/')
      }
      ListFiles(para).then(response => {
        this.folderList = response.data.Directories
        this.filesList = response.data.Files
      })
    },
    // 刷新当前列表
    refreshGetList() {
      this.queryParams = this.currentLocationName;
      this.getList()
      this.$message({
        message: '已经成功获取最新数据啦！',
        type: 'success'
      });
      this.initClickId()
    },

    //上传文件
    addFile() {
      if (this.currentLocationName === "edudoc") {
        this.$message({
          message: "根目录不允许上传文件！",
          type: 'error'
        });
        return
      }
      this.$refs.addFolder1.open();
    },
    // 下载文件
    down(item) {
      let para = {
        FileName: item,
        savePath: ""
      }
      DownloadFile(para).then(res => {
        if (res.status === 200) {
          this.$message({
            message: res.data,
            type: 'success'
          });
        } else {
          this.$message({
            message: res.data,
            type: 'error'
          });
        }
        console.log(res);
      })
    },
    previewfile() {
      this.$message({
        type: 'warning',
        message: '该功能未开放'
      });
      return
      this.show = true
    },
    googleViewerUrl(url) {
      return `https://docs.google.com/gview?url=${encodeURIComponent(url)}&embedded=true`;
    },
    //创建文件夹
    addFolder() {
      if (this.currentLocationName === "edudoc") {
        this.$message({
          message: "根目录不允许创建文件夹！",
          type: 'error'
        });
        return
      }
      this.$prompt('请输入新文件夹名称', '创建文件夹', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
      }).then(({ value }) => {

        let para = {
          folderPath: this.currentLocationName === "edudoc" ? value : this.locationUrl.join('/') + "/" + value
        }
        CreateFolder(para).then(res => {
          if (res.status == 200) {
            this.$message({
              type: 'success',
              message: '创建成功 '
            });
            const that = this;
            setTimeout(function () {
              that.refreshGetList();  // 刷新当前页面
            }, 500);

          } else {
            this.$message({
              type: 'error',
              message: '创建失败 '
            });
          }
        })

      }).catch(() => {
      });
    },
    // 重命名文件夹
    updateFloder() {
      this.folderShow = false;
      this.$prompt('请输入文件夹的新名称', '重命名', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        inputValue: this.clickFolderName,
        inputErrorMessage: '输入不能为空',
        inputValidator: (value) => {       // 点击按钮时，对文本框里面的值进行验证
          if (!value) {
            return '输入不能为空';
          }
        },
      }).then(({ value }) => {
        let sysFolder = {
          folderName: value,
          folderId: this.clickFolderId //默认为0
        }
        renameFolder(sysFolder).then(res => {
          if (res.code == 200) {
            this.$message({
              type: 'success',
              message: '修改成功 '
            });
            let that = this;
            setTimeout(function () {
              that.refreshGetList();  // 刷新当前页面
            }, 500);

          } else {
            this.$message({
              type: 'error',
              message: '修改失败 '
            });
          }
        })

      })

    },
    //删除文件夹
    deleteFolder() {
      this.folderShow = false;
      let that = this;
      if (this.locationUrl.length === 0 || this.currentLocationName === "edudoc") {
        this.$message({
          type: 'error',
          message: '根目录文件夹禁止删除'
        });
        return
      }
      this.$confirm('此操作将永久删除该文件夹，包括文件夹内的所有内容，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let para = {
          folderPath: this.locationUrl.join('/') + "/" + this.clickFolderName
        }
        DeleteF(para).then(res => {
          if (res.status == 200) {
            this.$message({
              type: 'success',
              message: '删除成功 '
            });
            setTimeout(function () {
              that.refreshGetList();  // 刷新当前页面
            }, 1000);

          } else {
            this.$message({
              type: 'error',
              message: '删除失败！ '
            });
          }
        })

      })
    },
    //打开文件夹
    openFolder() {
      this.folderShow = false;
      this.queryParams = this.clickFolderName;
      this.locationUrl.push(this.currentLocationName)
      this.currentLocationName = this.clickFolderName;
      this.getList();
    },
    //鼠标双击文件夹
    doubleClickFolder(index, item) {
      this.clickFolderName = item;
      this.historyFolderName = this.currentLocationName
      this.currentLocationName = item;
      this.openFolder();
    },

    //文件夹右键
    rightClickFolder(index, item, e) {

      this.initClickId()

      this.clickFolderName = item
      this.folderStyle.left = e.pageX - 140 + 'px'
      this.folderStyle.top = e.pageY - 70 + 'px'
      this.folderShow = true

      this.moveData.typeofFolder = 1
    },
    //文件 右键
    rightClickfile(index, item, e) {
      this.initClickId()

      this.clickFileName = item
      this.clickFilePath = this.localBaseUrl + this.locationUrl.join('/') + "/" + item
      this.prefileUrl = this.baseUrl + this.locationUrl.join('/') + "/" + item
      this.fileStyle.left = e.pageX - 140 + 'px'
      this.fileStyle.top = e.pageY - 70 + 'px'
      this.fileShow = true


      this.moveData.typeofFolder = 2
    },
    //删除文件
    deleteFileFun() {
      this.fileShow = false;
      if (this.locationUrl.length === 0 || this.currentLocationName === "edudoc") {
        this.$message({
          type: 'error',
          message: '根目录文件禁止删除'
        });
        return
      }
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let para = {
          folderPath: this.locationUrl.join('/') + "/" + this.clickFileName
        }
        DeleteF(para).then(res => {
          if (res.status == 200) {
            this.$message({
              type: 'success',
              message: '删除成功 '
            });
            let that = this;
            setTimeout(function () {
              that.refreshGetList();  // 刷新当前页面
            }, 1000);

          } else {
            this.$message({
              type: 'error',
              message: '删除失败！ '
            });
          }
        })

      })

    },

    //移动文件(夹)
    moveFolder() {
      this.$message({
        type: 'warning',
        message: '该功能未开放'
      });
      return
      this.fileShow = false

      //通过判断文件/文件夹被右键选择而进行参数存储
      if (this.clickFolderId != -1) {
        this.moveData.clickFolderId = this.clickFolderId;
      } else {
        this.moveData.clickFolderId = this.clickFileId;
      }

      this.$refs.moveFolder1.open();
    },

    //初始化右键选择相关参数
    initClickId() {
      this.fileShow = false;
      this.folderShow = false;
    }

  },
  mounted() {
    //监听鼠标点击事件
    document.addEventListener("click", (e) => {
      if (!this.folderShow && !this.fileShow) return; // 如果右键菜单不显示，则不处理点击事件
      let target = e.target;
      while (target && target.parentNode) {
        if (target.parentNode.class === "folderContainer") {
          return;
        }
        target = target.parentNode;
      }
      this.folderShow = false;
      this.fileShow = false; // 如果点击的是其他区域，则隐藏
      this.clickFolderId = -1;
      this.clickFileId = -1;
    });
  },

  created() {
    this.getList()
  }
}
</script>


<style lang="less">
.pageHeader {
  .el-page-header__content {
    font-size: 16px !important;
  }
}

.iframeBox {
  position: fixed;
  top: 25%;
  left: 25%;
}
</style>


<style scoped lang="less">
.drawing_card {
  width: 100%;
  margin-top: 15px;
  transition: all 0.9s;
  border-radius: 10px;
  display: flex;
  flex-wrap: wrap;
  align-items: start;
}

.folderContainer {
  width: 150px;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  margin-left: 30px;
  height: auto;
}



.folder {
  width: 110px;
  height: 80px;
  perspective: 600px;
  transform-style: preserve-3d;
  cursor: pointer;
}

.folderWrapper {
  width: 140px;
  position: relative;
  transition: all .2s ease;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.folderWrapper:hover {
  background-color: aliceblue;
}


.folderName {
  margin-top: 5px;
  font-size: 14px;
  line-height: 20px;
  text-align: center;
}


.add-folder-9 {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 130px;
  box-shadow: 0px 0px 10px 0px rgb(149, 149, 150);
}

.add-folder-1 {
  overflow: hidden;
  width: 100%;
  height: 100%;
  background-color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.add-folder-2 {
  color: #19191a;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 25%;
  padding: 8px 0;

  a {
    text-decoration: none;
    color: #19191a;
  }
}

.add-folder-2:hover {
  background-color: rgba(6, 13, 20, .18);
  cursor: pointer;
}

.add-folder-6 {
  color: #19191a;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  padding: 8px 0;
  height: 25%;
}

.add-folder-6:hover {
  background-color: rgba(6, 13, 20, .18);
  cursor: pointer;
}
</style>
