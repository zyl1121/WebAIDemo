<script setup>
import UploadView from './components/UploadView.vue'
</script>

<template>
    <div class="container">
        <h1>林业有害生物智能识别</h1>
        <p>此网页用于测试使用WebAI.js调用PaddleClas导出的模型对林业有害生物进行智能识别。</p>
        <upload-view></upload-view>
    </div>
</template>

<style scoped>
.container {
    margin: 40px auto;
    max-width: 800px;
    max-height: 600px;
}
</style>
