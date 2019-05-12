<template>
    <div id="music">
        <div class="controls_left">
            <!--上一曲-->
            <a class="icon prev" @click="prevNext(true)"></a>
            <!--播放暂停-->
            <a class="icon play" :class="{pause:state.onOff}" @click="play"></a>
            <!--下一曲-->
            <a class="icon next" @click="prevNext(false)"></a>
        </div>
        <div class="albuimg">
            <img src="https://p3fx.kgimg.com/stdmusic/20190415/20190415183500348042.jpg"/>
        </div>
        <div class="schedule">
            <div class="song">
                <div class="song_name"><span>{{lists[index].name}}</span></div>
                <div class="duration">
                    <span>
                        <i>{{transformTime(tm_cur)}}</i>/<i>{{transformTime(tm_max)}}</i>
                    </span>
                </div>
            </div>
            <div class="bar">
                <progress :value="tm_cur" :max="tm_max" id="progress"></progress>
                <span class="playhead" :style="{left:tm_cur*100/tm_max+`%`}"></span>
            </div>
        </div>
        <div class="schedule_right">
            <div>
                <!--音量开关控制按钮-->
                <a class="icon maxvos" :class="{maxvos_on:state.maxvos,maxvos_off:!state.maxvos}" @click="volumeSwitch"></a>
                <!--音量控制面板-->
                <div class=""></div>
            </div>
            <div>
                <!--播放模式按钮-->
                <a class="icon cycle" :class="{cycle_1:state.mode==1,cycle_2:state.mode==2,cycle_3:state.mode==3}" @click="state.modeDis=!state.modeDis"></a>
                <div class="mode" v-show="state.modeDis">
                    <ul>
                        <li><a class="bot" :class="{active:state.mode==1}" @click="state.mode=1;state.modeDis=false;"><span class="icon loop"></span>列表循环</a></li>
                        <li><a class="bot" :class="{active:state.mode==2}" @click="state.mode=2;state.modeDis=false;"><span class="icon single"></span>单曲循环</a></li>
                        <li><a :class="{active:state.mode==3}" @click="state.mode=3;state.modeDis=false;"><span class="icon random"></span>随机播放</a></li>
                    </ul>
                </div>
            </div>
            <div>
                <!--下载按钮-->
                <a class="icon down" :download="lists[index].name" :href="lists[index].url"></a>
            </div>
            <div>
                <!--分享按钮-->
                <a class="icon share"></a>
                <div class=""></div>
            </div>
            <div>
                <!--音乐列表按钮-->
                <a class="icon list"><span>{{lists.length}}</span></a>
                <!--音乐列表面板-->
                <div class=""></div>
            </div>
        </div>
        <audio :src="lists[index].url" id="audio" @playing="playing" @pause="state.onOff=false" @canplay="monitor" @ended="prevNext(false)">
            这个标签都不支持，我实在是没办法将就你
        </audio>
    </div>
</template>

<script>
    export default {
        name: "music",
        data(){
            return {
                lists:[{
                    name:"something just like",
                    url:"https://caihai123.github.io/applet/music/something%20just%20like%20this.mp3"
                },{
                    name:"三月烟花",
                    url:"https://caihai123.github.io/applet/music/三月烟花.mp3"
                },{
                    name:"歌曲1",
                    url:"https://caihai123.github.io/applet/music/歌曲1.mp3"
                }],
                audio:null,
                state:{
                    currentTime:null,
                    onOff:false,// false代表暂停true代表播放
                    mode:1,//1代表列表播放2代表单曲播放3代表随机播放
                    modeDis:false,//控制播放模式控制面板的显示和隐藏
                    maxvos:true//false代表静音
                },
                tm_max:0,//音频时长
                tm_cur:0,//当前音频播放进度
                index:0//当前播放音频的索引
            }
        },
        mounted:function () {
            this.audio = document.getElementById("audio");

        },
        methods:{
            play:function () {//播放and暂停
                if(this.audio.paused){this.audio.play();}else{this.audio.pause();}
            },
            prevNext:function(bo){//上一曲and下一曲
                switch(this.state.mode)
                {
                    case 1:
                        if(this.index==(bo?0:this.lists.length-1)){
                            this.index = bo?this.lists.length-1:0;
                        }else{
                            bo?this.index--:this.index++;
                        }
                        break;
                    case 2:
                        this.audio.currentTime = 0;
                        break;
                    case 3:
                        var that = this;
                        var n;
                        var f = function(){
                            n = Math.floor(Math.random()*that.lists.length);
                            if(n==that.index){
                                f();
                            }else{
                                that.index = n;
                            }
                        }
                        f();
                      break;
                    default:
                        console.log("应该不会执行到这里")
                }
                this.state.onOff = false;
                this.audio.autoplay = true;
            },
            monitor:function () {//当音频准备就绪后获取音频时长
                this.tm_max = this.audio.duration;
            },
            playing:function(){//音频开始播放时执行的函数
                var that = this;
                that.state.onOff=true;
                that.currentTime = setInterval(function(){
                    that.tm_cur = that.audio.currentTime;
                },500)
            },
            pause:function(){//音频暂停时执行的函数
                this.state.onOff=false;
                clearInterval(this.currentTime);
            },
            transformTime:function (tm) {//将时长改为00:00格式的字符
                let time = parseInt(tm);
                let m = time/60<10?`0${parseInt(time/60)}`:parseInt(time/60);
                let s = time%60<10?`0${time%60}`:time%60;
                return `${m}:${s}`
            },
            volumeSwitch:function () {//开启或关闭静音的函数
                this.audio.muted = !this.audio.muted;
                this.state.maxvos = !this.state.maxvos;
            },
            download:function () {
                window.location.href = this.lists[this.index].url;
            }
        },
        computed:{
            
        },
        watch:{

        }
    }
</script>

<style scoped>
    ul,li{margin: 0;padding: 0;list-style: none;}
    #music{min-width: 894px;height: 80px;position: absolute;bottom: 0;left: 0;width: 100%;display: flex;align-items:center;justify-content:center;background: #000;color: #fff;}
    i{font-style: normal;}
    #music .icon{display: block;cursor:pointer;background-image: url("../assets/btn.png");background-repeat: no-repeat;}

    .controls_left{width: 160px;height: 80px;display: flex;padding:0 7px;align-items:center;}
    .controls_left .prev{width: 36px;height: 36px;background-position:0 -143px;}
    .controls_left .prev:hover{background-position: -36px -143px;}
    .controls_left .play{width: 60px;height: 60px;margin:0 7px;}
    .controls_left .pause{background-position: 0 -60px;}
    .controls_left .next{width: 36px;height: 36px;background-position:-144px -143px;}
    .controls_left .next:hover{background-position: -180px -143px;}

    .albuimg{width: 60px;height: 60px;}
    .albuimg img{width: 60px;height: 60px;}

    .schedule{width: 370px;height: 80px;padding-left: 20px;box-sizing: border-box;}
    .schedule .song{height: 24px;margin-top:18px;display: flex;font-size: 13px;line-height:24px;}
    .schedule .song .song_name{width: 290px;}
    .schedule .song .duration{width:80px;text-align: right;}
    .schedule .bar{position: relative;}
    .schedule .bar progress{width: 100%;height: 3px;}
    .schedule .bar .playhead{display: block;width: 12px;height: 12px;position: absolute;top:12px;border-radius: 6px;background-color:#fff;}

    .schedule_right{width: 260px;height: 80px;display: flex;align-items:center;justify-content:space-between;margin-left:30px;position: relative;}
    .schedule_right .maxvos{width: 16px;height: 16px;}
    .schedule_right .maxvos_on{background-position: -64px -195px;}
    .schedule_right .maxvos_on:hover{background-position: -80px -195px;}
    .schedule_right .maxvos_off{background-position: -128px -195px;}
    .schedule_right .maxvos_off:hover{background-position: -144px -195px;}
    .schedule_right .cycle{width: 16px;height: 16px;}
    .schedule_right .cycle_1{background-position: -64px -179px;}
    .schedule_right .cycle_1:hover{background-position: -80px -179px;}
    .schedule_right .cycle_2{background-position: 0 -179px;}
    .schedule_right .cycle_2:hover{background-position: -16px -179px;}
    .schedule_right .cycle_3{background-position: -128px -179px;}
    .schedule_right .cycle_3:hover{background-position: -144px -179px;}

    .schedule_right .down{width: 15px;height: 15px;background-position: -240px -32px;}
    .schedule_right .down:hover{background-position: -256px -32px;}
    .schedule_right .share{width: 15px;height: 15px;background-position: -240px 0;}
    .schedule_right .list{width: 60px;height: 23px;background-position: 0 -120px;text-align: center;font-size: 13px;}

    .schedule_right .mode{width: 100px;height: 90px;position: absolute;margin-left: -50px;margin-top: -161px;padding: 10px;z-index: 999;font-size: 13px;background: #262b33;}
    .schedule_right .mode li{height: 30px;line-height: 30px;}
    .schedule_right .mode li a{display: block;width: 100%;height: 100%;cursor: pointer;}
    .schedule_right .mode li .bot{border-bottom: 1px solid #30343d;}
    .schedule_right .mode li .active{color: #19b5f0;}
    .schedule_right .mode li a span{width: 16px;height: 16px;margin:7px 8px;float: left;}
    .schedule_right .mode .loop{background-position: -64px -179px;}
    .schedule_right .mode li .active .loop{background-position: -96px -179px;}
    .schedule_right .mode .single{background-position: 0 -179px;}
    .schedule_right .mode li .active .single{background-position: -32px -179px;}
    .schedule_right .mode .random{background-position: -128px -179px;}
    .schedule_right .mode li .active .random{background-position: -160px -179px;}
</style>