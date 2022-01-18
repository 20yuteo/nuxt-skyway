<template>
  <section class="container">
    <div>
      <video id="their-video" width="200" autoplay playsinline></video>
      <video id="my-video" width="400px" autoplay muted playsinline></video>

      <div class="main">
        <h2>Nuxt.js + SkyWayのビデオチャット</h2>
        マイク:
        <select v-model="selectedAudio" @change="onChange">
          <option disabled value="">Please select one</option>
          <option v-for="(audio, key, index) in audios" v-bind:key="index" :value="audio.value">
            {{ audio.text }}
          </option>
        </select>

        カメラ: 
        <select v-model="selectedVideo" @change="onChange">
          <option disabled value="">Please select one</option>
          <option v-for="(video, key, index) in videos" v-bind:key="index" :value="video.value">
            {{ video.text }}
          </option>
        </select>

        <div>
          <p>Your id: <span id="my-id">{{peerId}}</span></p>
          <p>他のブラウザでこのIDをコールしましょう。</p>
          <h3>コールする</h3>
          <input v-model="calltoid" placeholder="call id">
          <button @click="makeCall" class="button--green">Call</button>
        </div>
      </div>

    </div>
  </section>
</template>

<script lang="js">
import Vue from 'vue'
import Peer from 'skyway-js'

export default {
  data() {
    return {
      APIKey: 'ef2b5f2e-5575-49e5-9a34-0756bb8a90ac',
      selectedAudio: '',
      selectedVideo: '',
      audios: [],
      videos: [],
      localStream: null,
      peerId: '',
      calltoid: ''
    }
  },

    methods: {
    onChange: function () {
      console.log(this.selectedAudio)
      console.log(this.selectedVideo)
      if(this.selectedAudio != '' && this.selectedVideo != ''){
        console.log('dsfsa')
        this.connectLocalCamera();
      }
    },

    connectLocalCamera: async function(){
      const constraints = {
        audio: this.selectedAudio ? { deviceId: { exact: this.selectedAudio } } : false,
        video: this.selectedVideo ? { deviceId: { exact: this.selectedVideo } } : false
      }

      const stream = await navigator.mediaDevices.getUserMedia(constraints);
      const videoEl = document.getElementById('my-video')
      videoEl.srcObject = stream
      videoEl.play()
      // document.getElementById('my-video').srcObject = stream;
      this.localStream = stream;
    },

    makeCall: function () {
      console.log('makeCall');
      const call = this.peer.call(this.calltoid, this.localStream);
      this.connect(call);
    },

    connect: function (call) {
      call.on('stream', stream => {
        console.log('test')
        console.log(stream)
        const el = document.getElementById('their-video');
        el.srcObject = stream;
        el.play();
      });
    }
  },

  mounted: async function () {
    this.peer = new Peer({
      key:   this.APIKey,
      debug: 3,
    });

    this.peer.on('open', () => {
      this.peerId = this.peer.id
    });

    this.peer.on('call', call => {
      call.answer(this.localStream);
      this.connect(call);
    });

    //デバイスへのアクセス
    const test = await navigator.mediaDevices.getUserMedia( {video: true,audio: true,})
    console.log(test)
    await navigator.mediaDevices.enumerateDevices()
      .then((deviceInfos) => {
        console.log(deviceInfos)
        for (let i = 0; i !== deviceInfos.length; ++i) {
          const deviceInfo = deviceInfos[i]
          if (deviceInfo.kind === 'audioinput') {
            this.audios.push({
              text: deviceInfo.label ||
              `Microphone ${this.audios.length + 1}`,
              value: deviceInfo.deviceId
            })
          } else if (deviceInfo.kind === 'videoinput') {
            this.videos.push({
              text: deviceInfo.label ||
              `Camera  ${this.videos.length - 1}`,
              value: deviceInfo.deviceId
            })
          }
        }
      })

  }
}
</script>
