<template>
  <div id="app">
    <div style="width: 50%; margin: 0 auto;">
      <h3>蜻蜓FM下载：</h3>
      <el-form ref="form" :model="form" label-width="120px">
        <el-form-item label="下载蜻蜓">
          <el-input type="input" v-model="form.qtUrl" placeholder="请输入蜻蜓FM页面URL"></el-input>
        </el-form-item>
      </el-form>
      <el-button type="primary" @click="downloadQT">下载（新打开页面）</el-button>

      <el-divider></el-divider>

      <h3>网易云音乐下载：</h3>
      <el-form ref="form" :model="form" label-width="120px">
        <el-form-item label="网易云音乐地址">
          <el-input type="input" v-model="form.music163Url" placeholder="请输入网易云音乐地址"></el-input>
        </el-form-item>
      </el-form>
      <el-button type="primary" @click="downloadMusic163">下载（新打开页面）</el-button>
    </div>
  </div>
</template>

<script>
import createHmac from 'create-hmac';

export default {
  name: 'app',
  data() {
    return {
      form: {
        qtUrl: '',
        music163Url: ''
      }
    }
  },
  methods: {
    downloadQT() {
      const urlObj = new URL(this.form.qtUrl);
      const match = /\/channels\/(\d+)\/programs\/(\d+)\//.exec(urlObj.pathname);
      const channelId = match && match[1];
      const programId = match && match[2];
      const downloadUrl = this.getRedirectUrl(channelId, programId);
      this.download(downloadUrl);
    },
    downloadMusic163() {
      const urlObj = new URL(this.form.music163Url);
      const match = /#\/song\?id=(\d+)/.exec(urlObj.hash);
      const id = match && match[1];
      this.download(`http://music.163.com/song/media/outer/url?id=${id}.mp3`);
    },
    download(url) {
      const a = document.createElement('a');
      a.download = url;
      a.href = url;
      a.target = '_blank';
      a.innerText = '点击下载';
      a.click();
    },
    getRedirectUrl(channelId, programId) {
      var authuser = '';
      var qingtingId = '';
      if (channelId && programId) {
          var params = {
              authuser: authuser,
              device_id: 'MOBILESITE',
              qingting_id: qingtingId,
              t: Date.now()
          };

          var searchParams = new URLSearchParams();
          Object.keys(params).forEach(key => searchParams.append(key, params[key]));
          var query = `?${searchParams.toString()}`;
          var path = `/audiostream/redirect/${channelId}/${programId}/${query}`;
          var sign = createHmac('md5', 'fpMn12&38f_2e').update(path).digest('hex').toString();
          return  `https://audio.qingting.fm${path}&sign=${sign}`;
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
