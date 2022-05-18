<template>
    <div id="deskBasic" ref="bg" :style="{backgroundImage : 'url(' + bgImg + ')',backgroundRepeat: 'no-repeat',
backgroundSize:['100%','100%'],height:bodyHeight,width: bodyWidth}">
        <el-container>
            <el-header :height="headerHeight">
                <span style="float:left;">活动</span>
               
                <div class="weather-date">
                  <!--时间、日历实现 -->
                    <el-popover
                      placement="bottom"
                      width="400"
                      trigger="click">
                      <!-- <H5Calendar lunar :touchthreshold="50"/> -->
                       <H5Calendar
                          lunar
                          :touchthreshold="50"
                          isshowtoday
                        />
                      <span class="weather-font"  slot="reference">{{dateTime}}</span>
                    </el-popover>
                   <!-- <div style="width:10rem"></div> -->
                   <!-- <el-divider direction="vertical"></el-divider> -->
                   <!--  单日天气 、七日天气-->
                      <el-popover
                        placement="bottom"
                        width="480"
                        trigger="click">
                        <Weather></Weather>
                        <div slot="reference" class="weather-state">
                          <span>{{weather.city}}</span>
                          <el-divider direction="vertical"></el-divider>
                          <el-image :src="WeatherImg" fit="scale-down" class="weather-img"></el-image>
                          <span class="weather-font">
                              {{weather.wea}}
                          </span>
                          <el-divider direction="vertical"></el-divider>
                          <span class="weather-font">今日气温：{{weather.tem_night+'°'+' ~ '+weather.tem_day+'°'}}</span>
                      </div>
                    </el-popover>


                </div>

                <el-tooltip class="item" effect="dark" content="点击更改壁纸" placement="bottom">
                  <span class="mouse"  @click="dialogVisible = true" circle><i class="el-icon-edit"></i></span>
                </el-tooltip>
                
            </el-header>
            <el-container>
                <el-aside :width="asideWidth">
                  <Software @dblclick.native="createTermView"></Software>
                  <FileSoft @dblclick.native="openFile"></FileSoft>
                  <SystemShow @dblclick.native="openSystemShow"></SystemShow>
                </el-aside>
                <el-container>
                    <el-main ref="main" id="test">
                      <File v-if="fileShow"></File>
                      <SystemState v-if="isSystemShow"></SystemState>
                      <!-- <test-term></test-term> -->
                      <!-- <console-window></console-window>     -->
                      <el-dialog
                            ref="end"
                            title="主题"
                            :visible.sync="dialogVisible"
                            width="30%"
                            :modal="false"
                            class="theme">
                            <Theme  @getTheme="getImg"></Theme>
                      </el-dialog>
                    </el-main>
                </el-container>
            </el-container>
        </el-container>
    </div>
</template>

<script>
import dayjs from "dayjs"
import Software from '../components/Software.vue'
import ConsoleWindow from '../components/ConsoleWindow.vue'
import Tab from '../components/Tab.vue'
import Theme from '../components/Theme.vue'
import store from '../store'
import Vue from 'vue'
import FileSoft from '../components/FileSoft.vue'
import File from '../components/File.vue'
import TestTerm from '../components/TestTerm.vue'
import Weather from '../components/Weather.vue'
import H5Calendar from "vue-calendar-h5";
import SystemShow from '../components/SystemShow.vue'
import SystemState from '../components/SystemState.vue'
    export default {
        name:'DeskTest',
        data(){
          return {
            dateTime:"",
            timer:null,
            headerHeight:window.innerHeight*0.04+'px',
            asideWidth:window.innerWidth*0.05+'px',
            footerHeight:window.innerHeight*0.06+'px',
            bodyHeight:window.innerHeight+'px',
            bodyWidth:window.innerWidth+'px',
            dialogVisible: false,
            bgImg:require('../assets/desktop.png'),
            fileShow:false,
            isSystemShow:false,
            weather:{},
            WeatherImg:'',
            zIndexMax:1,  //  设置一个图层属性，最低是1，
            gridData:[]
          }
        },
        // computed: {
        //     termShow() {
        //         return Vue.state.termShow
        //     },
        //     tabShow() {
        //         return Vue.state.tabShow
        //     }
        // },
        components:{Software,ConsoleWindow,Tab,Theme,FileSoft, File, TestTerm, Weather,H5Calendar, SystemShow,SystemState},
        created() {
          
          
          // console.log('屏幕可用高宽 '+screen.availHeight+' '+ screen.availWidth);
          // console.log('屏幕高宽 '+screen.Height+' '+ screen.Width);
          console.log('浏览器高宽 '+window.outerHeight+" "+ window.outerWidth)
          console.log('浏览器可用高宽 '+window.innerHeight+" "+ window.innerWidth)
          // console.log('body宽高 '+body.offsetHeight+" "+body.offsetWidth);
          // console.log('body宽高 '+body.clientHeight+" "+body.clientWidth);
        },
        methods:{
          getImg(value){
            
            // 更改背景图片
            this.bgImg = value
            // this.$refs.bg.style.backgroundImage = value
            // console.log(this.$refs.bg.style)
            
            // console.log(value)
          },
          createTermView(){
            // console.log("创建了一个term终端")
            // this.$refs.main.$el.innerHTML('<ConsoleWindow/>');
            // let term = "<ConsoleWindow/>"
            // for (const iterator of this.$refs.main.$el.children) {
            //   console.log(iterator);
            // }
            // 获取到当前的main下的所有div的数量
            let childrenCount = this.$refs.main.$el.children.length
            // 创建要挂载的div视图
            let term= document.createElement("div")
            term.id = 'terminal'
            // 插入到相对应的位置
            this.$refs.main.$el.insertBefore(term,this.$refs.end.$el)
            
            // Vue.component('console-window',)
            // console.log(this.$refs.main.$el.location);
            // document.getElementById("test").innerHTML(term)
            // this.$refs.main.innerHTML('<ConsoleWindow/>')
            // 创建一个组件实例，并挂载到对应的视图上
            var tt = Vue.extend(TestTerm)
            new tt().$mount('#terminal')
            
            // 配置多窗口的位置和图层问题
            let termCount = childrenCount - 1 
            let terms = this.$refs.main.$el.children
            let thisConsole = terms[termCount]

            thisConsole.style.zIndex = this.zIndexMax > childrenCount ? this.zIndexMax + 1 : childrenCount
            this.zIndexMax = parseInt(thisConsole.style.zIndex)
            // this.$refs.main.$el.children[heightIndex-1].style.zIndex = heightIndex + 1
            // this.$refs.main.$el.children[heightIndex-1].style.left = (280 + (heightIndex-1) * 76.58)+'px'
            // this.$refs.main.$el.children[heightIndex-1].style.top= (49 + (heightIndex-1) * 32)+'px'
            
            thisConsole.style.left = (280 + (termCount) * 76.58)+'px'
            thisConsole.style.top= (49 + (termCount) * 32)+'px'
            // 绑定点击事件

            thisConsole.onclick = e => {
              
              let currentZIndex = parseInt(thisConsole.style.zIndex)
              // console.log("点击前当前最高层："+this.zIndexMax);
              if(currentZIndex != this.zIndexMax){
                thisConsole.style.zIndex = this.zIndexMax + 1
                this.zIndexMax++
              }
            }
            // console.log("创建完毕")
            // 设置图层
          },
          openFile(){
            this.fileShow = !this.fileShow
          },
          openSystemShow(){
            this.isSystemShow = !this.isSystemShow
          },
          getWeatherImg(weather){
                let imgSrc = weather.wea_img
                if(weather.wea_img == 'qing' || weather.wea_img == 'yun'){
                    if(parseInt(dayjs().format('HH'))  > 18){
                        imgSrc+='night'
                    }
                }
                this.WeatherImg = require('../assets/'+imgSrc+'.png')
          }
        },
        mounted() {
          // 开启一个定时器，使页面的时间能够实时变化
          this.timer = setInterval(() => {
              this.dateTime = dayjs().format("YYYY年MM月DD日 dddd HH:mm:ss")
          },144)
          if(localStorage.getItem('bgImg') != null){
            this.bgImg = localStorage.getItem('bgImg')
          }
          // 请求天气接口
          this.$axios.get('https://yiketianqi.com/free/day',{
                params:{
                    appid:'34962419',
                    appsecret:'GhNEd877',
                    vue:'1'
                }
            }).then(res => {
                this.weather = res.data
                this.getWeatherImg(this.weather)
            })
        },
        updated(){
            
        }, 
        beforeDestroy() {
            //  销毁前关闭定时器
            clearInterval(this.timer)
        }
    }
</script>

<style scoped>
  #deskBasic{
    /* height: 722px;
    width:1536px; */
    /* background-image: url('../assets/desktop.png'); */
  }
  .el-container{
    padding: 0px;
    margin: 0px;
    height: 100%;
    /* opacity: 0.9; */
  }
  .el-header{
    background-color: #232425;
    color: rgb(233, 224, 224);
    text-align: center;
    height: 1.5%;
    opacity: 0.4;
    border-radius:5px;
    display: flex;
    justify-content:space-between;
    align-items: center;
  } 
  .el-footer {
    background-color: #242629;
    color: #333;
    padding: 0px;
    /* text-align: center; */
    opacity: 0.4;
    border-radius:5px;
    display: flex;
    justify-content: flex-start;
    flex-direction: row;
    /* text-align: center;
    align-items: center; */
  }
  
  .el-aside {
    background-color: #373738;
    color: #333;
    text-align: center;
    opacity: 0.75;
    border-radius:5px;
  }
  .el-main {
    /* color: #333; */
    display: flex;
    justify-content: flex-start;
    flex-direction: column;
    text-align: center;
    align-items: center;
  }
  span{
    font-size: 1.5rem;
    /* font-size: larger; */
  }
  .mouse{

    /* background-color: aqua; */
    width: 40px;
  }
  .mouse:hover{
    background-color: #333;
  }
  .weather-font{
      font-size: 1.5rem;
  }
  .weather-state{
      display: flex;
      align-items: center;
      margin-left: 10rem;
  }
  .weather-date{
      display: flex;
      align-items: center;
      
  }
  .weather-img{
      width: 24px;
      height: 24px;
  }
</style>