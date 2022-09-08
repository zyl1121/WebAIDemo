<script setup>
import { ref } from 'vue'
// import { UploadFilled } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'
import { UploadFilled } from '@element-plus/icons-vue'
import WebAI from 'webai-js'
import { onMounted } from '@vue/runtime-core'
WebAI.ort.env.wasm.wasmPaths = "https://cdn.jsdelivr.net/npm/webai-js@1.1.4/dist/"
globalThis.cv = WebAI.cv

let model = {};
const imgDom = ref(null);
const tableData = ref([])
const imageUrl = ref(null);
const modelURL = './mobilenet_v3/model.onnx'
const modelConfig = './mobilenet_v3/configs.json'
const loading = ref(false)

onMounted(async () => {
    console.log(imgDom.value);
    try {
        // 设置onnxruntime-web wasm路径
        model = await WebAI.Cls.create(modelURL, modelConfig)
        ElMessage({
            message: '模型加载成功!',
            type: 'success',
        })
    } catch {
        ElMessage({
            message: '模型加载失败!',
            type: 'error',
        })
    }
})

const f = async () => {
    let imgRGBA = cv.imread(imgDom.value);
    loading.value = true;
    tableData.value = [];
    setTimeout(async () => {
        try {
            // 获取置信度
            let probs = await model.infer(imgRGBA)

            // 显示识别结果
            // textProbs.innerHTML = JSON.stringify(probs, null, 4)
            // console.log(JSON.stringify(probs, null, 4))
            tableData.value = probs;
            console.log(tableData.value);

            ElMessage({
                message: '识别成功，下拉查看结果！',
                type: 'success',
            })
        }
        catch {
            ElMessage({
                message: '识别失败!',
                type: 'success',
            })
        }
        finally {
            loading.value = false
            imgRGBA.delete();    // 删除图片对象
        }
    }, Math.random() % 500 + 700);
}

const handleChange = (uploadFile, _) => {
    tableData.value = [];
    const fileTypes = ["image/jpeg", "image/jpg", "image/png"];
    if (fileTypes.includes(uploadFile.type)) {
        ElMessage({
            message: '格式不正确!',
            type: 'error',
        })
    } else {
        imageUrl.value = URL.createObjectURL(uploadFile.raw)
        ElMessage({
            message: '上传成功!',
            type: 'success',
        })
    }
}
const clear = () => {
    console.log("clear");
    imageUrl.value = '';
    imgDom.value = null;
    tableData.value = [];
}

</script>

<template>
    <!-- <div v-show="imageUrl" style="text-align: center;"> </div> -->
                
    <el-upload class="upload-demo" drag :auto-upload="false" :show-file-list="false" :on-change="handleChange">

        <div v-show="!imageUrl" class="el-upload__text"
            style="height: 400px; width: 600px; align-text: center; margin:auto; display：flex vertical-align: middle;">
            <div style="padding: 130px;">
                <el-icon v-show="!imageUrl" :size="80" style="align-text: center;  ">
                    <UploadFilled />
                </el-icon>
                <div style="align-text: center; vertical-align: middle; margin: 8px;">
                    拖拽文件到此或 <em style="color:#409eff;">点击上传</em>
                </div>
            </div>
        </div>

        <template #tip>
            <div class="el-upload__tip" style="font-size: 14px;">
                请上传jpg或png格式文件
            </div>
        </template>
        <img ref="imgDom" :src="imageUrl" id="imgDom" class="imgDom" />
    </el-upload>

    <div v-show="imageUrl" style="text-align: center; padding: 10px; margin: 10px">
        <el-button type="primary" @click="f" style="text-align: center; vertical-align: top;">开始识别</el-button>
        <el-button @click="clear" style="text-align: center; vertical-align: top;">清空结果</el-button>
    </div>

    <div v-if="tableData">
        <h2>识别结果</h2>
        <el-table v-loading="loading" :data="tableData.splice(0, 4)" stripe style="width: 100%;"
            :header-cell-style="{'text-align':'center'}" :cell-style="{'text-align':'center'}">
            <el-table-column prop="label" label="名称" />
            <el-table-column prop="prob" label="置信度" :formatter="formatNum" />
        </el-table>
    </div>

</template>

<style scoped>
.imgDom {
    max-height: 400px;
    text-align: center;
    margin: 0;
    padding: 0;
}

.el-upload {
    max-height: 600px;
}

.el-upload-dragger .el-upload__text {
    font-size: 20px;
}

.el-table .v-align-t {
    vertical-align: top;
}
</style>