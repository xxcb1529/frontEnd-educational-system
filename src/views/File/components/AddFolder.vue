<template>
    <el-dialog v-if="dialogVisible" :modal-append-to-body="false" :close-on-click-modal="false" title="上传文件"
        :visible.sync="dialogVisible" :show-close="false" width="400px">
        <el-upload ref="upload" action="none" class="upload-demo" drag :on-success="uploadSuccess"
            :on-error="uploadError" :on-change="handleChange" :file-list="fileList" :auto-upload="false" multiple
            :http-request="uploadFile">
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将文件拖到此处，或<em>点击选择文件</em></div>
        </el-upload>
        <div style="margin-top: 20px">
            <el-button icon="el-icon-upload2" type="success" @click="submit">提交</el-button>
            <el-button @click="close">取消</el-button>
        </div>
    </el-dialog>
</template>
<script>

import { UploadSingleFile, UploadMultipleFile } from "../../../api/api";
export default {
    name: 'addFolder',
    props: ['locationUrl'], //当前所处文件夹id
    data() {
        return {
            dialogVisible: false,
            fileList: []
        }
    },
    methods: {
        open() {
            this.dialogVisible = true
        },
        close() {
            this.dialogVisible = false
            this.$parent.refreshGetList();
        },
        //上传成功
        uploadSuccess(res) {
            this.dialogVisible = false
            this.$parent.refreshGetList();
        },
        //上传失败
        uploadError(err) {
            console.log(err);
            this.$message.error('服务器异常请重试!');
        },
        handleChange(file, fileList) {
            this.fileList = fileList;
        },
        //上传文件
        submit() {
            this.$refs.upload.submit();
            this.fileList = []
        },
        check(fileName) {
            let type = fileName.slice(fileName.lastIndexOf('.') + 1);
            var fileType = {
                "pdf": 1,
                "docx": 2,
                "ppt": 3
            };
            return fileType[type];
        },
        // 上传
        uploadFile(files) {
            let that = this
            const loading = this.$loading({
                lock: true,
                text: '上传中...',
                spinner: 'el-icon-loading',
                background: 'rgba(0, 0, 0, 0.7)'
            });
            return new Promise((resolve, reject) => {
                // if (this.fileList && this.fileList.length <= 1) {
                let data = new FormData();
                data.append('FileDetails', files.file);
                data.append('FileType', this.check(files.file.name));
                data.append('folderPath', this.locationUrl);
                UploadSingleFile(data).then(res => {
                    if (res.status == 200) {
                        this.$message({
                            type: 'success',
                            message: '上传成功 '
                        });
                        loading.close();
                        that.dialogVisible = false
                        setTimeout(()=>{
                            that.$parent.refreshGetList();
                        },500)
                        resolve()
                    } else {
                        this.$message({
                            type: 'error',
                            message: '上传失败 '
                        });
                        reject()
                    }
                })
                // } 
                // else {
                //     let fileArr = []
                //     this.fileList.forEach((fileItem) => {
                //         let file = new FormData();
                //         file.append('FileDetails', fileItem.raw);
                //         file.append('FileType', this.check(fileItem.raw.name));
                //         file.append('folderPath', this.locationUrl);
                //         fileArr.push(file)
                //     });
                //     UploadMultipleFile(fileArr).then(res => {
                //         console.log(res);
                //     })
                // }
            })
        },
    }
}
</script>
<style scoped lang="less"></style>
