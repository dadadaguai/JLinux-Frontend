<template>
    <div class="bottomBody">
        <div class="loginCard">
                <div class="imgPos">
                    <el-image
                        style="width: 8rem; height: 8rem"
                        :src="require('../assets/logo.png')">
                    </el-image>            
                </div>   
                <el-divider content-position="center">JLinux-Login</el-divider>
                <el-form ref="form" :model="form" label-width="240px">
                    <el-input v-model="form.account" placeholder="用户名"></el-input>
                    <el-input v-model="form.password" type="password" placeholder="密码"></el-input>
                    <div class="operatePasswd">
                        <div>
                            <span>记住密码 </span>
                            <el-switch v-model="form.record"></el-switch>
                        </div>
                        <!-- <el-link class="pos">忘记密码？</el-link> -->
                    </div>
                    <el-button type="primary" :plain="true" @click="onSubmit" round>登录</el-button>
                </el-form>
        </div>
    </div>
 
</template>

<script>
import axios from 'axios';
    export default {
        name:'Login',
        data() {
            return {
                form: {
                    account: '',
                    password: '',
                    record: false,
                },
            }
        },
        methods: {
            onSubmit() {
                if(this.form.account.length === 0 || this.form.password.length === 0){
                    this.$message.error('账号密码不能为空，请输入账号密码，再点击登录。');
                }else{
                    // 如果用户点击了记住密码选项，则将密码账户缓存到用户本地。
                    if(this.form.record){
                        localStorage.setItem('userInfo',JSON.stringify(this.form))
                    }else{
                        localStorage.setItem('userInfo',JSON.stringify({
                            account: '',
                            password: '',
                            record: false,
                        }))
                    }
                    var user = this.form.account
                    // 与后端进行密码验证，并返回用户信息
                    axios.post('api/login',{
                        username: this.form.account,
                        password: this.form.password
                    }).then(res => {
                        var userInfo = {
                            user:user,
                            path:res.data.path,
                            token:res.data.token
                        }
                        this.$store.commit('setUserInfo',userInfo)
                    })
                    // 页面跳转
                    this.$router.push({
                        name:'Desktop'
                    })
                    // 并将输入框内恢复成默认状态
                    this.form ={
                            account: '',
                            password: '',
                            record: false,
                    }
                }
            }
        },
        mounted() {
            // 先判断本地缓存里是否有用户信息，若有，则直接绑定到用户密码上。
            if(localStorage.length !=0){
                if(localStorage.getItem('userInfo') !== null){
                    this.form = JSON.parse(localStorage.getItem('userInfo'))
                }else {
                    this.form = {
                            account: '',
                            password: '',
                            record: false,
                    }                    
                }
            }else {
                this.form = {
                            account: '',
                            password: '',
                            record: false,
                }
            }
        }
    }
</script>

<style>
    .bottomBody{
        height: 824px;
        display: flex;
        flex-direction: column;
        align-items:center;  
        justify-content: center;  
        /* background-color: aquamarine;   */
        background-image: url('../assets/desktop.png');  
    }
    .loginCard{
        height: 420px;
        width:360px;
        border-radius:5px;
        background-color:rgb(237, 245, 245);
        opacity: 0.8;
        display: flex;
        flex-direction: column;
        justify-content:flex-start;
        align-items:center;
    }
    .el-input{
        margin-top: 0.5rem;
        margin-bottom: 0.5rem;
        width: 300px;
    }
    .el-form{
        display: flex;
        flex-direction: column;
        justify-content:flex-start;
        align-items:center;
        /* background-color: aqua; */
    }
    .operatePasswd{
        width: 100%;
        margin-top: 0.2rem;
        margin-bottom: 0.5rem;
        display: flex;
        flex-direction: row;
        justify-content:space-between;
    }
    .el-button{
        margin-top: 20px;
        width: 240px;
    }
    .imgPos{
        display: flex;
        flex-direction: column;
        justify-content:flex-start;
        align-items:center;
    }
</style>