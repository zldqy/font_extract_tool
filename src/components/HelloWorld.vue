<template>
  <div style="min-width: 800px;">
    <Spin :loading="state.running" dot>
      <h2>中莱达企业字体抽取工具</h2>
      <p style="width: 350px; text-align: left; margin: auto; margin-bottom: 8px; text-indent:2em;">
        该工具用于从一个字体文件中提取指定的文字，导出的字体中将只包含你需要的文字。
      </p>
      <div style="display: flex; align-items: center; width: 400px; margin: auto;">
        <span style="width: 200px; text-align: right;">请选择字体源文件（.ttf）：</span>
        <Upload style="width: 150px;" :show-file-list="false" :custom-request="uploadIt" accept=".ttf" />
      </div><br>
      当前字体：{{ state.oriFontName }}
      <div v-show="state.oriFont" style="display: flex; align-items: center; width: 400px; margin: auto; justify-content: center;">
        源字体预览：<h3>
          我能吞下玻璃而不伤身体
        </h3>
      </div>
      <div style="text-align: left;" v-show="state.oriFont">
        需要抽取的文字：
        <Space direction="vertical" size="large" fill>
          <Textarea style="height: 300px;" placeholder="赵钱孙李周吴郑王" v-model="state.inputText" default-value=""
            allow-clear />
          <div style="max-width: 70vw; word-break: break-all;">预览：<h3 style="margin-top: 0;">{{ state.inputText }}</h3>
          </div>
        </Space>
        <Button type="primary" @click="exportIt">导出字体</Button>&nbsp;
        <Button @click="loadInput2500">2500个常用字</Button>&nbsp;
        <Button @click="clearInput">清空</Button>
      </div>
    </Spin>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { Upload, Space, Textarea, Button, Spin } from '@arco-design/web-vue'
import { Font } from 'fonteditor-core';

const state = reactive({
  oriFont: null,
  oriFontName: '未选择字体',
  running: false,
  inputText: ''
})

const uploadIt = (e) => {
  state.oriFont = e.fileItem.file
  state.oriFontName = e.fileItem.file.name
  document.head.innerHTML += `<style>@font-face {font-family: 'oriFont'; src: url(${URL.createObjectURL(e.fileItem.file)});} h3 {font-family: 'oriFont';}</style>`
}

const clearInput = () => {
  state.inputText = ''
}

const loadInput2500 = () => {
  state.running = true
  let request = new XMLHttpRequest()
  request.open("get", "./dict.json")
  request.send()
  request.onload = (dict) => {
    let json = JSON.parse(request.responseText)
    state.inputText = json["2500"]
    state.running = false
  }
}

const exportIt = () => {
  state.running = true
  const oriReader = new FileReader();
  oriReader.readAsArrayBuffer(state.oriFont);
  oriReader.onloadend = (buffer) => {
    const oriFont = Font.create(buffer.target.result, {
      type: 'ttf',
      hinting: true
    });
    const text = document.querySelector('textarea').value
    const textUniList = text.split('').map(e => e.codePointAt(0))
    const result = oriFont.find({
      unicode: textUniList
    });
    const targetBuffer = undefined;
    const targetFont = Font.create(targetBuffer, {
      type: 'ttf',
      hinting: true
    });
    const oriFontObject = oriFont.get();
    const targetFrontObject = targetFont.get();
    Object.keys(oriFontObject).forEach(key => {
      if (key !== 'glyf') {
        targetFrontObject[key] = oriFontObject[key];
      }
    });
    targetFrontObject.glyf = [
      ...targetFrontObject.glyf,
      ...result
    ];
    const tempLink = document.createElement('a');
    tempLink.download = state.oriFontName.split('.')[0] + '_output.ttf'
    tempLink.href = targetFont.toBase64({ type: 'ttf' })
    tempLink.click()
    state.running = false
  }
}

</script>

<style scoped>

</style>
