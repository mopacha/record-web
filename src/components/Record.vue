<template>
  <div class="record">
    <input class="input" type="text" /><br />

    <button @click="start" :disabled="disabled.start">开始录制</button>
    <button @click="stop" :disabled="disabled.stop">结束录制</button>
    <button @click="download" :disabled="disabled.download">下载文件</button>
    <a id="download"></a>
  </div>
</template>

<script>
export default {
  name: "Record",
  props: {
    msg: String,
  },

  data() {
    return {
      // 本地流
      stream: null,
      // 媒体录制
      mediaRecorder: null,
      // 数据块
      chunks: [],
      // 录制结果
      recording: null,
      // 按钮禁用
      disabled: {
        start: false,
        stop: true,
        download: true,
      },
    };
  },

  methods: {
    // 获取屏幕分享的权限
    openScreenCapture() {
      if (navigator.getDisplayMedia) {
        return navigator.getDisplayMedia({ video: true });
      } else if (navigator.mediaDevices.getDisplayMedia) {
        return navigator.mediaDevices.getDisplayMedia({ video: true });
      } else {
        return navigator.mediaDevices.getUserMedia({
          video: { mediaSource: "screen" },
        });
      }
    },

    // 开始屏幕分享录制
    async start() {
      this.disabled.start = true;
      this.disabled.stop = false;
      this.disabled.download = true;

      if (this.recording) {
        window.URL.revokeObjectURL(this.recording);
      }

      // 获取屏幕分享权限
      this.stream = await this.$options.methods.openScreenCapture();
      // 实例化一个MediaRecorder对象
      this.mediaRecorder = new MediaRecorder(this.stream, {
        mimeType: "video/webm",
      });
      // 监听可用数据
      this.mediaRecorder.addEventListener("dataavailable", (event) => {
        if (event.data && event.data.size > 0) {
          this.chunks.push(event.data);
        }
      });
      // 开始录制
      this.mediaRecorder.start(10);
    },

    // 停止屏幕分享录制
    stop() {
      this.disabled.start = true;
      this.disabled.stop = true;
      this.disabled.download = false;

      // 停止录制
      this.mediaRecorder.stop();
      // 释放MediaRecorder
      this.mediaRecorder = null;
      // 停止所有流式视频轨道
      this.stream.getTracks().forEach((track) => track.stop());
      // 释放getDisplayMedia或getUserMedia
      this.stream = null;

      // 获取当前文件的一个内存URL
      this.recording = window.URL.createObjectURL(
        new Blob(this.chunks, { type: "video/webm" })
      );
    },

    // 下载录制的视频内容
    download() {
      this.disabled.start = false;
      this.disabled.stop = true;
      this.disabled.download = true;

      const downloadLink = document.querySelector("a#download");
      downloadLink.href = this.recording;
      // download 规定作为文件名来使用的文本
      downloadLink.download = "screen-recording.webm";
      downloadLink.click();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.record {
  input{
    width: 300px;
    margin-bottom: 20px;
  }
  button {
    margin: 0 1em 1em 0;
    padding: 0.5em 1.2em 0.6em 1.2em;
    border: none;
    border-radius: 4px;
    background-color: #d84a38;
    font-family: "Roboto", sans-serif;
    font-size: 0.8em;
    color: white;
    cursor: pointer;
  }
  button:hover {
    background-color: #c03434;
  }
  button[disabled] {
    background-color: #c03434;
    pointer-events: none;
  }
}
</style>
