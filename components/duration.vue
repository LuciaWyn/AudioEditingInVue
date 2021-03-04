<template>

</template>

<script>
import axios from 'axios'

export default {
    name: "duration",
    props: ['audio'],
    data(){
        return{
            audiotime: 0
        }
    },
    mounted(){
        const conf = {
            responseType: 'arraybuffer',
            onDownloadProgress: this.downloadProgress
        }
        axios.get(this.audio, conf)
            .then(response => this.decode(response))
            .catch(err => {
                console.error(`Failed to get file '${this.audio}'`)
                console.log(err)
            })
    },
    methods:{
        decode(response){
            const ctx = new AudioContext()
            ctx.decodeAudioData(response.data, (audioBuffer) => {
                this.setPeaks(audioBuffer)
            }, (err) => {
                console.error('Failed to decode audio data.')
                console.log(err)
            })
        },
        setPeaks: function (buffer) {
            this.audiotime = buffer.duration;
            this.$emit('audiotime', buffer.duration);
        }
    }
}
</script>

<style scoped>

</style>
