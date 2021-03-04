<template>
    <div>
        <av-waveform
            :width="widthy"
            :canv-width="parseInt(widthy)"
            :canv-height="parseInt(heighty)"
            :height="heighty"
            :audio-src="audioSrc"
            class="audiowav"
        >

        </av-waveform>

        <duration :audio="audioSrc" @audiotime="time" />

        <div class="blank" ref="blank" >


            <vue-resizable ref="vr" style="position: absolute;" :min-height="200" :height="200" :max-height="200" width="1010px" :max-width="widthit" :left="12"
                           @mount="eHandler"
                           @resize:move="eHandler"
                           @resize:start="eHandler"
                           @resize:end="eHandler2"
                           @drag:move="eHandler"
                           @drag:start="eHandler"
                           @drag:end="eHandler"
            >
                <div class="audioselect">
                </div>
            </vue-resizable>
        </div>
        <div>
            <p>Volume percentage: {{volumepercent}}%</p>
            <input type="range" @change="sendout" class="form-control-range" min="1" max="100" v-model="volumepercent">
        </div>

        <button v-if="playing==false" @click="audioy">Play Edited Audio</button>
    </div>
</template>

<script>
import AvWaveform from "./AvWaveform.js";

import VueResizable from 'vue-resizable';

import Duration from "./duration";

export default {
    name: "audioblocks",
    props:["audioSrc"],
    components:{
        Duration,
        AvWaveform,
        VueResizable
    },
    data(){
        return{
            widthy: '1000px',
            heighty: '200px',
            widthit: 1015,
            resizing: {
                left: 0,
                right: 0
            },
            audioduration: 0,
            audiostart: 0,
            audiofinish: 0,
            volumepercent: 100,
            playing: false,
        }
    },
    methods:{
        eHandler(data) {
            let left = data.left-12;
            let right = 1015-((left)+data.width);
            let leftposition;
            let rightposition;
            let leftleft = left;
            let rightleft = left+data.width-10;

            if((left== 0)&&(right ==0)){
                this.widthit = 1021;
            }
            else{
                if((left == this.resizing.left)){
                    leftposition = 'still';
                }
                else if(left > this.resizing.left){
                    leftposition = 'right';
                }
                else if(left < this.resizing.left){
                    leftposition = 'left';
                }
                if((right ==this.resizing.right)){
                    rightposition = 'still';
                }
                else if(right > this.resizing.right){
                    rightposition = 'left';
                }
                else if(right < this.resizing.right){
                    rightposition = 'right';
                }
                if((leftposition == 'left')&&(rightposition =='still')){
                    this.widthit = 1021-right;
                }
                else if((rightposition == 'right')&&(leftposition =='still')){
                    this.widthit = 1021-data.left;
                }
            }
            this.resizing.left = left;
            this.resizing.right = right;

            if(leftleft <=0){
                this.audiostart = 0;
            }
            else{
                this.audiostart =(leftleft/1000)*this.audioduration;
            }
            if(rightleft >=1000){
                this.audiofinish = this.audioduration;
            }
            else{
                this.audiofinish = (rightleft/1000)*this.audioduration;
            }

            this.sendout()
        },
        eHandler2(data){
            this.widthit = 1021;

            let left = data.left-12;
            let leftleft = left;
            let rightleft = left+data.width-10;


            if(leftleft <=0){
                this.audiostart = 0;
            }
            else{
                this.audiostart =(leftleft/1000)*this.audioduration;
            }
            if(rightleft >=1000){
                this.audiofinish = this.audioduration;
            }
            else{
                this.audiofinish = (rightleft/1000)*this.audioduration;
            }
            this.sendout()
        },
        time(dur){
            this.audioduration = dur;
            this.audiofinish = dur;

            this.sendout()
        },
        audioy(){
            let aud = new Audio()
            aud.src = this.audioSrc
            aud.currentTime = this.audiostart;
            let adur = this.audiofinish*1000;
            adur = Math.floor(adur);
            aud.volume = this.volumepercent/100;
            this.playing = true;
            aud.play();
            setTimeout(()=> {
                aud.pause();
                this.playing = false;
            }, adur);

            this.sendout()
        },
        sendout(){
            let end = this.audiofinish*1000;
            end = Math.floor(end);
            let start = this.audiostart*1000;
            start = Math.floor(start);
            let data = {volume:(this.volumepercent/100), start: start, finish: end};
            this.$emit('sendback', data);
        }
    }
}
</script>

<style scoped>
.blank{
    width: 1010px;
    height: 210px;
    position: relative;
    margin-top:-210px;
}
.audioselect{
    width:100%;
    height:100%;
    border-left:5px solid blue;
    border-right:5px solid cyan;
}
.audiowav{
    margin-left:15px;
}
button{
    padding: 0px;
    background-color: #f8fafc;
    border: none;
}
</style>
