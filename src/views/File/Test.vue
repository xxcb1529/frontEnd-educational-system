<template>
    <div>
        <input type="text" v-model="inputUrl" placeholder="接口地址" />
        <input type="text" v-model="params" placeholder="参数" />
        <input type="file" name="" id="" @change="handleFileUpload" multiple>
        <button @click="sendG">Get请求</button>
        <button @click="sendP">Post请求</button>
        {{ res }}
    </div>
</template>
<script>

import {
    testGetApi,
    testPostApi
} from "../../api/api";
export default {
    data() {
        return {
            res: {},
            inputUrl: "",
            params: "",
            filesToUpload: []
        }
    },
    methods: {
        handleFileUpload(event) {
            this.filesToUpload = event.target.files;
        },
        sendG() {
            let para = {
                userId: this.params
            }
            testGetApi(this.inputUrl, para).then(res => {
                this.res = res.data
                console.log(res);
            })
        },
        sendP() {
            const formData = new FormData();
            formData.append('file', this.filesToUpload[0]); 
            formData.append('folderPath', this.params);
            console.log(formData);
            let para = {
                folderPath: this.params,
                file: this.filesToUpload
            }
            testPostApi(this.inputUrl, formData).then(res => {
                this.res = res.data
                console.log(res);
            })
        }
    }
}
</script>