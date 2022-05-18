<template>
    <el-container>
        <el-header height="40px">
            <span style="margin-bottom:1rem"> 
                {{"cpu的"+cpu[0]}}
            </span>
            <span >
                {{cpu[1]+" , "+cpu[4]+" , "+cpu[5]}}
            </span>
        </el-header>
        <el-main>
            <div ref="chart" id="chart" style="width: 100%; height:360px; overflow:hidden;"  ></div>
        </el-main>
    </el-container>
</template>

<script>
export default {
    name:'CPU',
    data(){
        return {
            userInfo: {
                user: "",
                path: "",
                token: "",
            },
            cpu:[],
            cpuOption:{
                systemUsed:0,
                userUsed:0,
                wait:0,
                idle:0
            }
        }
    },
    watch:{
        cpu:{
            handler(newCpu,oldCpu){
                this.init()
            },
            deep:true,
        }
    },
    methods:{
        initData(){
            this.login()
            this.getData()

        },
        login() {
            this.$axios.post("api/login", {
                username: "root",
                password: "123456",
            }).then((res) => {
                //  获取后台数据
                this.userInfo.user = "root",
                this.userInfo.path = res.data.path,
                this.userInfo.token =  res.data.token
            })
        },
        getData(){
            this.$axios.post('api/exec',{
                token: this.userInfo.token,
                command: "cpu",
                time: new Date().getMilliseconds(),
            }).then(
                res => {
                    this.cpu = res.data.res
                    let cpuOption = res.data.res
                    this.cpuOption.systemUsed = parseFloat(cpuOption[2].split('率')[1].split('%')[0])
                    this.cpuOption.userUsed = parseFloat(cpuOption[3].split('率')[1].split('%')[0])
                    this.cpuOption.wait =  parseFloat(cpuOption[4].split('率')[1].split('%')[0])
                    this.cpuOption.idle =  parseFloat(cpuOption[5].split('率')[1].split('%')[0])
                    console.log(this.cpuOption);
                }
            )
        },
        init(){
            var chartDom = this.$refs.chart;
            var myChart = this.$echarts.init(chartDom);
            var option = {
                title: {
                    text: 'CPU运行状态',
                    left: 'center'
                },
                tooltip: {
                    trigger: 'item'
                },
                legend: {
                    orient: 'vertical',
                    left: 'left'
                },
                series: [
                    {
                    name: 'CPU运行状态',
                    type: 'pie',
                    radius: '50%',
                    data: [
                        { value: this.cpuOption.systemUsed, name: 'CPU系统使用率' },
                        { value: this.cpuOption.idle, name: 'CPU当前空闲率' },
                        { value: this.cpuOption.userUsed, name: 'CPU用户使用率' },
                        { value: this.cpuOption.wait, name: 'CPU当前等待率' },
                    ],
                    emphasis: {
                        itemStyle: {
                        shadowBlur: 10,
                        shadowOffsetX: 0,
                        shadowColor: 'rgb(201, 201, 201)'
                        }
                    }
                    }
                ]
            }
            
            myChart.setOption(option);
        }
    },
    mounted(){
        this.initData()
    }
}
</script>

<style scoped>
    .el-header{
         /* background-color: rgb(201, 201, 201); */
         display: flex;
         flex-direction: column;
         font-size: 1.2rem;
         color: aliceblue;
    }
</style>