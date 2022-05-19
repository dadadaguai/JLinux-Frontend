<template>
  <div ref="console" class="term">
     <el-upload v-show="false"
                class="upload-demo"
                ref="upload"
                action="api/upload"
                :file-list="fileList"
                :on-change="handleChange"
                :on-success="uploadSuccess"
                :data="fileInfo"
                :auto-upload="true">
            <el-button slot="trigger" size="small" type="primary" ref="select" v-show="false">选取文件</el-button>
      </el-upload>
  </div>
</template>

<script>
import { Terminal } from 'xterm';
import { FitAddon } from 'xterm-addon-fit';
import { AttachAddon } from 'xterm-addon-attach'
import "xterm/css/xterm.css";
import axios from 'axios';
import Vue from 'vue';
	export default {
    name:"Term",
    data() {
      return {
        fileList: [],
        fileInfo:{
            time:new Date().getTime(),
            token:'',
            command: '',
        },
        userInfo: {
          user: "",
          path: "",
          token: "",
        },
        textColor:{
          black: '\x1b[30m',
          red: '\x1b[31m',
          green: '\x1b[32m',
          yellow: '\x1b[33m',
          blue: '\x1b[34m',
          magenta: '\x1b[35m',
          cyan: '\x1b[36m',
          white: '\x1b[37m',
          brightBlack: '\x1b[1;30m',
          brightRed: '\x1b[1;31m',
          brightGreen: '\x1b[1;32m',
          brightYellow: '\x1b[1;33m',
          brightBlue: '\x1b[1;34m',
          brightMagenta: '\x1b[1;35m',
          brightCyan: '\x1b[1;36m',
          brightWhite: '\x1b[1;37m'
        },
        commandList:[
            {
              "cmd": "cat",
              "format": "cat [-AbeEnstTuv] fileName",
              "params": [
                "-n 由1开始对所有输出的行数编号。",
                "-b 和-n相似，只不过对于空白行不编号。",
                "-s 当遇到有连续两行以上的空白行，就代换为一行的空白行。",
                "-v 使用 ^ 和 M- 符号，除了 LFD 和 TAB 之外。",
                "-E 在每行结束处显示 $。",
                "-T 将 TAB 字符显示为 ^I。",
                "-A 等价于 -vET。",
                "-e 等价于\"-vE\"选项；",
                "-t 等价于\"-vT\"选项；"
              ],
              "desc": "用于连接文件并打印到标准输出设备上",
              "example": ""
            },
            {
              "cmd": "cd",
              "format": "cd [dirName]",
              "params": [
                "~ 表示为 home 目录 的意思",
                ". 则是表示目前所在的目录",
                ".. 则表示目前目录位置的上一层目录"
              ],
              "desc": "用于切换当前工作目录",
              "example": ""
            },
            {
              "cmd": "chmod",
              "format": "chmod [-cfvR] file...",
              "params": [
                "-c 若该文件权限确实已经更改，才显示其更改动作",
                "-f 若该文件权限无法被更改也不要显示错误讯息",
                "-v 显示权限变更的详细资料",
                "-R 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)"
              ],
              "desc": "控制用户对文件的权限的命令。",
              "example": ""
            },
            {
              "cmd": "clear",
              "format": "clear",
              "params": [],
              "desc": "清除当前终端显示的所有内容",
              "example": ""
            },
            {
              "cmd": "cp",
              "format": "cp [options] source [dest|directory]",
              "params": [
                "-f 覆盖已经存在的目标文件而不给出提示",
                "-p 除复制文件的内容外，还把修改时间和访问权限也复制到新文件中。",
                "-i 与 -f 选项相反，在覆盖目标文件之前给出提示，要求用户确认是否覆盖，回答 y 时目标文件将被覆盖。",
                "-r 若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。"
              ],
              "desc": "主要用于复制文件或目录",
              "example": ""
            },
            {
              "cmd": "cpu",
              "format": "cpu",
              "params": [],
              "desc": "用于显示cpu的当前状态",
              "example": ""
            },
            {
              "cmd": "find",
              "format": "find path -option [-print] [ -exec -ok command] {} ;",
              "params": [
                "-time 在过去多少毫秒内被修改过的文件",
                "-e 空的文件",
                "-p 路径名称符合 p 的文件",
                "-name 文件名称符合 name 的文件(即以递归的方式逐个变更)",
                "-size 文件大小是n单位",
                "-p 权限查询",
                "-g 查询用户文件"
              ],
              "desc": "用来在指定目录下查找文件",
              "example": ""
            },
            {
              "cmd": "grep",
              "format": "grep [-bcdFGHilLnov][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>] [范本样式][文件或目录...]",
              "params": [
                "-A <显示行数>,除了显示符合范本样式的那一列之外，并显示该行之后的内容。",
                "-b 在显示符合样式的那一行之前，标示出该行第一个字符的编号。",
                "-B <显示行数>,除了显示符合样式的那一行之外，并显示该行之前的内容。",
                "-R 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)",
                "-c 计算符合样式的列数。",
                "-C <显示行数>,除了显示符合样式的那一行之外，并显示该行之前后的内容。",
                "-d <动作>,当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。",
                "-F 将样式视为固定字符串的列表。",
                "-G 将样式视为普通的表示法来使用。",
                "-H 在显示符合样式的那一行之前，表示该行所属的文件名称。",
                "-i 忽略字符大小写的差别。",
                "-l 列出文件内容符合指定的样式的文件名称。",
                "-L 列出文件内容不符合指定的样式的文件名称。",
                "-n 在显示符合样式的那一行之前，标示出该行的列数编号。",
                "-o 只显示匹配PATTERN 部分。",
                "-v 显示不包含匹配文本的所有行。"
              ],
              "desc": "用于查找文件里符合条件的字符串",
              "example": ""
            },
            {
              "cmd": "help",
              "format": "help [-dmsp] [PATTERN...]",
              "params": [
                "-d 输出每个命令的简短描述",
                "-a 输出命令的所有内容",
                "-s 只显示命令使用格式",
                "-p 显示命令可使用的参数"
              ],
              "desc": "用于显示内部命令的帮助信息",
              "example": ""
            },
            {
              "cmd": "jvm",
              "format": "jvm",
              "params": [],
              "desc": "用于显示JVM的当前状态",
              "example": ""
            },
            {
              "cmd": "ls",
              "format": "ls [params] [pathName]",
              "params": [
                "-a 显示所有文件及目录 (.开头的隐藏文件也会列出)",
                "-l 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出",
                "-A 同-a,但不列出 . (目前目录) 及 .. (父目录)",
              ],
              "desc": "用于显示指定工作目录下之内容（列出目前工作目录所含之文件及子目录)。",
              "example": ""
            },
            {
              "cmd": "Mem",
              "format": "Mem",
              "params": [],
              "desc": "用于显示内存的当前状态",
              "example": ""
            },
            {
              "cmd": "mkdir",
              "format": "mkdir [-p] dirName",
              "params": [
                "-p 确保目录名称存在，不存在的就建一个"
              ],
              "desc": "用于创建目录",
              "example": ""
            },
            {
              "cmd": "mv",
              "format": "mv [options] source [dest|directory]",
              "params": [
                "-b 当目标文件或目录存在时，在执行覆盖前，会为其创建一个备份。",
                "-i 如果指定移动的源目录或文件与目标的目录或文件同名，则会先询问是否覆盖旧文件，输入 y 表示直接覆盖，输入 n 表示取消该操作。",
                "-f 如果指定移动的源目录或文件与目标的目录或文件同名，不会询问，直接覆盖旧文件。",
                "-n 不要覆盖任何已存在的文件或目录。",
                "-u 当源文件比目标文件新或者目标文件不存在时，才执行移动操作"
              ],
              "desc": "用来为文件或目录改名、或将文件或目录移入其它位置",
              "example": ""
            },
            {
              "cmd": "pwd",
              "format": "pwd",
              "params": [],
              "desc": "用于显示工作目录",
              "example": ""
            },
            {
              "cmd": "rm",
              "format": "rm [options] name...",
              "params": [
                "-i 删除前逐一询问确认",
                "-f 即使原档案属性设为唯读，亦直接删除，无需逐一确认",
                "-r 将目录及以下之档案亦逐一删除"
              ],
              "desc": "用于删除一个文件或者目录",
              "example": ""
            },
            {
              "cmd": "touch",
              "format": "touch [-cm][-d<日期时间>][-r<参考文件或目录>] [文件或目录…]",
              "params": [
                "-m 改变档案的修改时间记录。",
                "-c 假如目的档案不存在，不会建立新的档案",
                "-d 设定时间与日期，可以使用各种不同的格式",
                "-r 使用参考档的时间记录，与 --file 的效果一样"
              ],
              "desc": "用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。",
              "example": ""
            }
        ],
        cmdStorage:[],
        storageIndex:0,
        term:Object,
        fitAddon:Object,
        cmd:'',
      };
    },
    methods:{
      // 上传文件时前的操作
      handleChange(file, fileList){
        console.log(file,fileList)
        this.fileInfo.time = new Date().getTime()
        this.fileInfo.token = this.userInfo.token
        this.fileInfo.command = 'rz'
      },
      // 上传成功后的操作
      uploadSuccess(response, file, fileList){
        this.term.writeln(this.textColor.brightYellow+"文件上传成功")
        this.cmd = "";
        this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
      },
      //处理字符的输入，如控制台常有的root@path:$
      strHandle() {
          return arguments.length == 2 ? arguments[0] + '@' + arguments[1] + ':$ ' : arguments[0] + '\n'
      },
      login() {
          axios.post("api/login", {
            username: "root",
            password: "123456",
          }).then((res) => {
            console.log(res);
            //  获取后台数据
            this.userInfo.user = "root",
            this.userInfo.path = res.data.path,
            this.userInfo.token =  res.data.token
            // xterm 进行渲染交互,ref属性绑定到相对应的dom上
            // this.term.open(this.$refs.console);
            this.term.write(this.textColor.brightWhite+this.strHandle(this.userInfo.user,this.userInfo.path));
            console.log(this.userInfo.path);
            // 自适应屏幕大小
            this.fitAddon.fit();
          })
      },
      exec() {
        // 记录命令
        this.cmd = ""
        // 监听键盘输入事件
        this.term.onKey((e) => {
        /*
          esc = 27
          回车 = 13
          上下左右 = 37,38,39,40
          backspace = 8
          向上箭头  38
          向下箭头  40
        */
        let code = e.domEvent.which;
        if (code === 13) {
          this.term.write(this.strHandle(e.key))
          //  去除左方空格
          this.cmdStorage.push(this.cmd)
          let cmds = this.cmd.split(' ')
          if(cmds[0] == ''){
            this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
          }else if(cmds[0] == 'help'){
              // help 无其他参数，格式 help
              if(cmds.length === 1){
                this.commandList.forEach(
                  res => {
                    this.term.writeln(this.textColor.yellow+res.cmd+":"+res.desc)
                    this.term.writeln("指令格式："+res.format)
                    res.params.forEach( 
                      param =>{
                        this.term.writeln("   "+param)
                    })
                })
                // 格式 help list  或者  help ls... 其他命令
              }else if(cmds.length === 2){
                if(cmds[1] == 'list'){
                  let len = this.commandList.length
                  for(let i = 0;i < len;++i){
                    if((i%5==0 && i != 0)){
                      this.term.write('\n')
                    }
                    this.term.write(this.textColor.yellow+this.commandList[i].cmd+"\t")
                    if(i == len-1){
                      this.term.write('\n')
                    }
                  }
                }else{
                  this.commandList.forEach(
                    res => {
                      if(res.cmd == cmds[1]){
                        this.term.writeln('\x1b[33m'+res.cmd+":"+res.desc)
                        this.term.writeln("指令格式："+res.format)
                        res.params.forEach( 
                          param =>{
                            this.term.writeln("   "+param)
                          })
                      }
                    }
                  )
                }
              // 格式 help -s ls  等等含有参数的查询
              }else{
                  this.commandList.forEach(
                    res => {
                    //-d 输出每个命令的简短描述",
                    //-a 输出命令的所有内容",
                    //-s 只显示命令使用格式",
                      if(res.cmd == cmds[2]){
                        let all = false
                        if(cmds[1] == '-a'){
                          all = true
                        }
                        if(all || cmds[1] == '-d'){
                          this.term.writeln(this.textColor.yellow+res.cmd+":"+res.desc)
                        }
                        if(all || cmds[1] == '-s'){
                          this.term.writeln(this.textColor.yellow+res.cmd+"指令格式："+res.format)
                        }
                        if(all || cmds[1] == '-p'){
                          res.params.forEach( 
                            param =>{
                              this.term.writeln(this.textColor.yellow+"   "+param)
                          })
                        }
                      }
                    }
                  )
              }
              this.cmd = "";
              this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
               
          // clear命令
          }else if(cmds[0] == 'clear'){
            if(cmds.length != 1){
              this.term.writeln("clear命令格式有误")
            }else{
              // 调用term的clear方法
              this.term.clear()
            }
            // 输出新的一行
            this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
            this.cmd = ""
          // 测试selectAll()方法
          }else if(cmds[0] == 'select'){
            this.term.selectAll()
            let strTemp = this.term.getSelection().replace(/[\r]/g, "").split("\n")
            let strList = strTemp.filter(res => {
              return res.length != 0
            })
            console.log(strList);
          }else if(cmds[0] == 'rz'){
            console.log("rz要被执行");
            this.$refs.select.$el.click()
          }else if(cmds[0] == 'sz'){
            if(cmds.length == 1){
              this.term.writeln(this.textColor.red+"命令输入错误")
              // 输出新的一行
              this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
              this.cmd = ""
            }else {
              let file = new File([],cmds[1])
              var formData = new FormData();
                formData.append("file",file,file.name)
                formData.append("token",this.userInfo.token)
                formData.append("time",new Date().getTime())
                formData.append("command",this.cmd)
                
                axios({
                  url:'api/upload',
                  method: 'post',
                  headers: {
                      'Content-Type': 'multipart/form-data'
                  },
                  data:formData
                }).then(
                  res => {
                    if(res.data.msg){
                      var arrayBuffer = new Uint8Array(res.data.bytS).buffer
                      const blob = new Blob([arrayBuffer])
                      const URL = window.URL
                      const href = URL.createObjectURL(blob)
                      const a = document.createElement('a')
                      a.href = href
                      a.download = res.data.name
                      document.body.appendChild(a)
                      a.click() //触发点击事件执行下载
                      a.style.display = 'none'
                      
                      const timer = setTimeout(function () {
                        a.remove() // document.body.removeChild(a)
                        // 释放blob对象
                        URL.revokeObjectURL(href)
                        clearTimeout(timer)
                      }, 1000)
                      this.term.writeln(this.textColor.green+"文件"+res.data.name+"下载完成。")
                      // 输出新的一行
                      this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
                      this.cmd = ""
                    }
                  }
                )
            }
          }else{
            axios
              .post("api/exec", {
                token: this.userInfo.token,
                command: this.cmd,
                time: new Date().getTime(),
              })
              .then((res) => {
                console.log(res);
                let command = this.cmd.split(" ")[0]
                
                if (command === "pwd") {
                  this.term.write('\x1b[1;33m'+this.strHandle(res.data.res[0]))
                } else if (command === "ls") {
                  // 遍历文件
                  var hasParams = this.cmd.split(" ").length == 1 // ls 无参数时为true
                  console.log(res.data.res);
                  res.data.res.forEach((item) => {
                      // 文本文件为绿色
                      var color = this.textColor.green
                      if(item.slice(0,1) == "d"){
                        // 目录为蓝色
                        color = this.textColor.brightCyan
                      }
                      if(hasParams){
                        this.term.write(color + item.split('\t')[1] +'\t')
                      }else{
                        this.term.writeln(color + item)
                      }
                  })
                  if(hasParams) this.term.write("\n")
                } else if (command === "cd") {
                  if(res.data.msg){
                    Vue.set(this.userInfo,"path",res.data.res[0])
                  }else{
                    this.term.writeln(res.data.res[0]+"文件")
                  }
                  
                }else if(command === "rm"){

                  this.term.writeln(res.data.res[0])
                }else if(command === 'Mem' || command === 'cpu' || command === 'jvm' || command === 'time' || command === 'systemFile'){
                  if(res.data.msg){
                    res.data.res.forEach(v => {
                      this.term.writeln('\t'+this.textColor.brightMagenta + v)
                      
                    })
                  }
                }else if(command === 'cat'){
                  let text = res.data.msg[0]
                  let len = text.length
                  for(let i =1;i<len;++i){
                    this.term.writeln(text[i]);
                  }
                }else if(command === 'find'){
                  res.data.res.forEach( v =>{
                    this.term.writeln(v)
                  })
                }
                this.cmd = "";
                this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
              }).catch(
                error => {
                  this.term.writeln(this.textColor.red+"命令输入错误")
                  this.cmd = "";
                  this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
                }
              )
          }
          this.storageIndex = this.cmdStorage.length
        } else if (code === 8) {
            if(this.cmd.length !== 0){
              this.term.write("\b \b")
              this.cmd = this.cmd.substring(0, this.cmd.length - 1)
            }
        } else if(code === 9){
          // 代码提示
          let arr = []
          this.commandList.forEach(
            v => {
                if(v.cmd.match(this.cmd) != null){
                  arr.push(v.cmd)
                }
            }
          )
          this.term.write('\n')
          for(let i = 0;i<arr.length;++i){
              if((i%5==0 && i != 0)){
                this.term.write('\n')
              }
              this.term.write(this.textColor.yellow+arr[i]+"\t")
              if(i == arr.length-1){
                this.term.write('\n')
              }
          }
          this.cmd = "";
          this.term.write(this.strHandle(this.textColor.brightWhite+this.userInfo.user,this.userInfo.path))
        }else if(code === 38){

          // 箭头向上代表上一个命令
          if(this.cmd != ''){
            let len = this.cmd.length
            for(var i=0;i<len;++i){
              this.term.write("\b \b")
            }
          }

          
          if(this.storageIndex == this.cmdStorage.length){
            this.storageIndex = this.storageIndex - 1
            this.cmd = this.cmdStorage[this.storageIndex]
            this.term.write(this.cmd)
          }else if(this.storageIndex == 0){
            this.cmd = this.cmdStorage[this.storageIndex]
            this.term.write(this.cmd)
          }else{
            this.storageIndex = this.storageIndex - 1
            this.cmd = this.cmdStorage[this.storageIndex]
            this.term.write(this.cmd)
          }
          console.log("箭头向上:"+this.storageIndex);
          console.log(this.cmdStorage);
          /**
           * 当index = len-1时，
           * 当index = 0 时，
           * 其他情况时 
           */
        }else if(code === 40){
          // 箭头向下代表下一个命令
          if(this.cmd != ''){
            let len = this.cmd.length
            for(var i=0;i<len;++i){
              this.term.write("\b \b")
            }
          }
          if(this.storageIndex == this.cmdStorage.length){
            this.cmd = ""
            this.term.write(this.cmd)
          }else if(this.storageIndex == 0){
            this.storageIndex = this.storageIndex + 1
            this.cmd = this.cmdStorage[this.storageIndex]
            this.term.write(this.cmd)
          }else if(this.storageIndex == this.cmdStorage.length -1){
            this.storageIndex = this.storageIndex + 1
            this.cmd = ""
            this.term.write(this.cmd)
          }else{
            this.storageIndex = this.storageIndex + 1
            this.cmd = this.cmdStorage[this.storageIndex]
            this.term.write(this.cmd)
          }

        }else if(code === 27){

        }else{
          this.cmd = this.cmd + e.key
          this.term.write(e.key)
          console.log(this.cmd);
        }
      });
      }
    },
    created() {
      this.term = new Terminal({
            rendererType: "canvas", //渲染类型
            rows: 28, //行数
            // cols: parseInt(_this.cols), // 不指定行数，自动回车后光标从下一行开始
            convertEol: true, //启用时，光标将设置为下一行的开头
            //   scrollback: 50, //终端中的回滚量
            disableStdin: false, //是否应禁用输入。
            cursorStyle: "underline", //光标样式
            cursorBlink: true, //光标闪烁
            windowsMode: false,
            // fontFamily: "Georgia", 字体样式
            // fontFamily: "Times New Roman", //字体样式,
            // fontFamily:"Droid Sans Mono",
            // fontSize:15,
            theme: {
              foreground: "rgb(255, 255, 255)", //字体，
              background: "#222222", //背景色
              cursor: "help", //设置光标
              lineHeight: 16,
            },
            
      });
      this.fitAddon = new FitAddon()
      this.term.loadAddon(this.fitAddon)
    },
    mounted() {
      this.term.open(this.$refs.console);
      this.login()
      this.exec()
    }
  }
</script>

<style>
  .test{
    background-color: rgb(255, 255, 255);
    background-color: #222222;
  }
</style>

