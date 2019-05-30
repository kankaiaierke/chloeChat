<template>
  <div class="index">
    <div class="fix-top">
      <el-row>
        <el-col :span="2">
          <i @click="backFunc" class="el-icon-arrow-left"></i>
        </el-col>
        <el-col :span="20" v-html="replyIng?'对方正在输入...':ai.name" class="textLeft">

        </el-col>
        <el-col :span="2">
          <i class="el-icon-more" v-if="false"></i>
        </el-col>
      </el-row>
    </div>
    <div class="fix-main">
      <ul>
        <li v-for="(item,index) in msgArr" :class="item.type">
          <span v-if="item.type=='chat-time'" v-html="item.msg"></span>
          <div v-if="item.type=='chat-receiver' || item.type=='chat-sender'">
            <div class="img">
              <el-image
                :src="item.head"
                :fit="'fill'"></el-image>
            </div>
          </div>
          <div v-if="item.type=='chat-receiver' || item.type=='chat-sender'" v-show="nameShow" v-html="item.name"></div>
          <div v-if="item.type=='chat-receiver' || item.type=='chat-sender'">
            <div :class="item.type=='chat-receiver'?'chat-right_triangle':'chat-left_triangle'"></div>
            <span v-html="item.msg"></span>
          </div>
        </li>
      </ul>
    </div>
    <div class="fix-buttom">
      <el-row>
        <el-col :xs="18" :sm="20" :md="22" :lg="22">
          <div class="grid-content bg-purple-dark">
            <el-input id="msgBox" :disabled="replyIng" v-model="input" placeholder="请输入内容" @keyup.enter.native="sendFunc"></el-input>
          </div>
        </el-col>
        <el-col :xs="6" :sm="4" :md="2" :lg="2">
          <el-button type="success" :disabled="replyIng || input == ''" @click="sendFunc">发送</el-button>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Index',
  data () {
    return {
      input: '',
      msgArr: [],
      user:{
        name: '蛋糕哥',
        head: 'http://himg.bdimg.com/sys/portrait/item/99896b616e6b6169313939307e02.jpg'
      },
      ai:{
        name: '周二珂',
        head: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1559125808876&di=626e9a1cb7c5fa0442724f87767e39c3&imgtype=0&src=http%3A%2F%2F02.imgmini.eastday.com%2Fmobile%2F20180224%2Fa7dfb7dcf14685c3ad4c525f94bb322b.jpeg'
      },
      dictionary: [
        {
          key: '我爱你',
          value: '蛋糕哥我也爱你（づ￣3￣）づ╭❤～'
        }
        ,{
          key: '你好',
          value: '蛋糕哥你好(｡･∀･)ﾉﾞ'
        }
        ,{
          key: '结婚',
          value: '讨厌，人家还是个孩子呐(ノω<。)ノ))☆.。'
        }
      ],
      replyIng: false,
      nameShow: false,
      lastTime: -1
    }
  },
  mounted () {
    document.getElementById('msgBox').focus()

  },
  methods: {
    backFunc: function () {
      this.$router.push({
        path: '/'
      })
    },
    sendFunc: function () {
      let self = this
      if(self.replyIng){
        return
      }
      if (self.input) {
        let nowTime = new Date().getTime()

        if (self.lastTime > 0 && nowTime <= self.lastTime + 300000) {

        } else {
          let timeMsg = self.GetDateAllStr('MM月dd日 hh:mm:ss')
          self.msgArr.push({
            msg: timeMsg,
            id: '',
            name: '',
            head: '',
            type: 'chat-time'
          })
        }
        self.lastTime = nowTime
        self.msgArr.push({
          msg: self.input,
          id: '',
          name: self.user.name,
          head: self.user.head,
          type: 'chat-receiver'
        })
        self.replyFunc(self.input)
      }
    },
    replyFunc: function (msg) {
      let self = this
      let reply = ''
      for (let i = 0 ; i < self.dictionary.length ; i++) {
        if (msg.indexOf(self.dictionary[i].key) >= 0) {
          reply = self.dictionary[i].value
          break
        }
      }
      self.replyIng = true
      setTimeout(() => {
        if (reply === '') {
          this.$get('http://open.drea.cc/chat/get?keyWord=' + self.input + '&userName=type%3Dbbs').then((res) =>{
            reply = res.data.data.reply
            while (reply.indexOf('MM') >= 0) {
              reply = reply.replace('MM','二珂')
            }
            self.doReply(reply)
          }).catch(function () {
            reply = '蛋糕哥加油↖(^ω^)↗'
            self.doReply(reply)
          })
          self.input = ''
        } else {
          self.doReply(reply)
          self.input = ''
        }
      }, reply.length * 100 )
    },
    doReply: function (reply) {
      var self = this
      self.msgArr.push({
        msg: reply,
        id: '',
        name: self.ai.name,
        head: self.ai.head,
        type: 'chat-sender'
      })
      self.replyIng = false
      setTimeout(() => {
        document.getElementById('msgBox').focus()
      },100)
    },
    GetDateAllStr:function (fmt, date) {
      if (fmt === undefined) fmt = 'yyyy-MM-dd hh:mm:ss';
      if (date === undefined) date = new Date();
      let o = {
        "M+": date.getMonth() + 1,                 //月份
        "d+": date.getDate(),                    //日
        "h+": date.getHours(),                   //小时
        "m+": date.getMinutes(),                 //分
        "s+": date.getSeconds(),                 //秒
        "q+": Math.floor((date.getMonth() + 3) / 3), //季度
        "S": date.getMilliseconds()             //毫秒
      };
      if (/(y+)/.test(fmt))
        fmt = fmt.replace(RegExp.$1, (date.getFullYear() + "").substr(4 - RegExp.$1.length));
      for (let k in o)
        if (new RegExp("(" + k + ")").test(fmt))
          fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
      return fmt;
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}
li {
  margin: 0;
  text-align: left;
}
a {
  color: #42b983;
}
.index{
  position: fixed;
  left:0;
  right:0;
  bottom:0;
  top:0;
  background: #eee;
}

.textLeft{
  text-align: left;
}

.fix-top{
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  border-bottom: 1px solid #ddd;
  font-size: 14px;
  line-height: 40px;
  height: 40px;
}

.fix-buttom{
  position: fixed;
  left:10px;
  right:10px;
  bottom:10px;
}
.fix-main{
  position: fixed;
  left:10px;
  right:10px;
  bottom:60px;
  top:40px;
  overflow: hidden;
  overflow-y: auto;
  padding-top: 10px;
}

.chat-sender{
  clear:both;
  font-size: 80%;
}
.chat-sender div:nth-of-type(1){
  float: left;
}
.chat-sender div:nth-of-type(2){
  margin: 0 50px 2px 50px;
  padding: 0px;
  color: #848484;
  font-size: 70%;
  text-align: left;
}
.chat-sender div:nth-of-type(3){
  background-color: white;
  /*float: left;*/
  margin: 0 50px 10px 50px;
  padding: 10px 10px 10px 10px;
  border-radius:7px;
  text-indent: -12px;
}

.chat-receiver{
  clear:both;
  font-size: 80%;
}
.chat-receiver div:nth-of-type(1){
  float: right;
}
.chat-receiver div:nth-of-type(2){
  margin: 0px 50px 2px 50px;
  padding: 0px;
  color: #848484;
  font-size: 70%;
  text-align: right;
}
.chat-receiver div:nth-of-type(3){
  /*float:right;*/
  background-color: #b2e281;
  margin: 0px 50px 10px 50px;
  padding: 10px 10px 10px 10px;
  border-radius:7px;
}

.chat-receiver div:first-child .img,
.chat-sender div:first-child .img{
  width: 40px;
  height: 40px;
  /*border-radius: 10%;*/
}

.chat-left_triangle{
  height: 0px;
  width: 0px;
  border-width: 6px;
  border-style: solid;
  border-color: transparent white transparent transparent;
  position: relative;
  left: -22px;
  top: 3px;
}
.chat-right_triangle{
  height: 0px;
  width: 0px;
  border-width: 6px;
  border-style: solid;
  border-color: transparent transparent transparent #b2e281;
  position: relative;
  right:-22px;
  top:3px;
}

.chat-notice,.chat-time{
  clear: both;
  font-size: 70%;
  text-align: center;
  margin-top: 15px;
  margin-bottom: 15px;
}

.chat-notice span{
  background-color: #cecece;
  line-height: 25px;
  border-radius: 5px;
  padding: 5px 10px;
  color: #fff;
}

.chat-time span{
  background-color: #eee;
  line-height: 25px;
  border-radius: 5px;
  padding: 5px 10px;
  color: #333;
}
</style>
