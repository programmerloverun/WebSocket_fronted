<template>
  <div class="wrapper">
    <div class="abs cover container">
      <div class="abs cover pnl">
        <div class="top pnl-head" style="padding: 20px ; color: white;">
          <div id="userName">
            当前用户：{{ username }}
            <span style="float: right;color: green;font-size: 20px" v-if="isOnline">在线</span>
            <span style="float: right;color: red;font-size: 20px" v-else>离线</span>
          </div>
          <div id="chatMes" v-show="chatMes" style="text-align: center;color: #6fbdf3;font-family: 新宋体,serif">
            正在和 {{ toName }} 聊天
          </div>
        </div>
        <!--聊天区开始-->
        <div class="abs cover pnl-body" id="pnlBody">
          <div class="abs cover pnl-left" id="initBackground" style="background-color: white; width: 100%">
            <div class="abs cover pnl-left" id="chatArea" v-show="isShowChat">
              <div class="abs cover pnl-message scroll" id="show">
                <div class="pnl-list" v-for="message in historyMessage">

                  <!-- 消息展示区域 -->
                  <!--对方在聊天框的左边-->
                  <div class="msg guest" v-if="message.fromName">
                    <div class="guest-name">{{ message.fromName }}</div>
                    <div class="msg-left">
                      <div class="msg-host headDefault"></div>
                      <div class="msg-ball">{{ message.message }}</div>
                    </div>
                  </div>
                  <!--自己在聊天框的右边-->
                  <div class="msg host" v-else>
                    <div class="hostname">
                      <p>{{ username }}</p>
                    </div>
                    <div class="msg-right">
                      <div class="msg-host photo">
                        <img src="@/img/avatar/Member001.jpg" alt="">
                      </div>
                      <div class="msg-ball">
                        {{ message.message }}
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <div class="abs bottom pnl-text">
                <div class="abs cover pnl-input">
                            <textarea class="scroll" id="context_text"
                                      @keyup.enter="submit" wrap="hard"
                                      placeholder="在此输入文字信息..."
                                      v-model="sendMessage.message">
                            </textarea>
                  <div class="abs auto-complete-pnl scroll hide">
                    <ul class="auto-complete"></ul>
                  </div>
                </div>

                <div class="abs br pnl-btn" id="submit" @click="submit"
                     style="background-color: rgb(32, 196, 202); color: rgb(255, 255, 255);">
                  发送
                </div>
              </div>
            </div>

            <!--聊天区 结束-->
            <div class="abs right pnl-right">
              <div class="slider-container hide"></div>
              <div class="pnl-right-content">
                <div class="pnl-tabs">
                  <div class="tab-btn active" id="hot-tab">好友列表</div>
                </div>
                <div class="pnl-hot">
                  <ul class="rel-list unselect">
                    <li class="rel-item" v-for="friend in friendsList"><a @click="showChat(friend)">{{ friend }}</a>
                    </li>
                  </ul>
                </div>
              </div>

              <div class="pnl-right-content">
                <div class="pnl-tabs">
                  <div class="tab-btn active">系统广播</div>
                </div>
                <div class="pnl-hot">
                  <ul class="rel-list unselect" id="broadcastList">
                    <li class="rel-item" v-for="name in systemMessages">
                      您的好友
                      {{ name }} 已上线
                    </li>
                  </ul>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {onMounted, ref} from 'vue'
import request from '@/util/request.js'
import {ElMessage} from "element-plus";

const webSocket = ref()

const isShowChat = ref(false)
const chatMes = ref(false)
const isOnline = ref(true)
const username = ref('')
const toName = ref('')
const sendMessage = ref({
  toName: '',
  message: ''
})
const historyMessage = ref([])
const friendsList = ref([])
const systemMessages = ref([])

onMounted(() => {
  init()
})

const init = async () => {
  await request
      .get('/user/getUsername')
      .then((response) => {
        console.log('获取用户名接口-后台返回的数据：', response)
        if (response.code === 200) {
          username.value = response.data
        }
      })
      .catch((error) => {
        console.log('获取用户名失败-错误信息：', error)
      })

  // 创建 WebSocket 对象
  webSocket.value = new WebSocket('ws://localhost:8080/chat')

  webSocket.value.onopen = onOpen

  // 接收到服务端推送的消息后触发
  webSocket.value.onmessage = onMessage

  webSocket.value.onclose = onClose
}

const onOpen = () => {
  isOnline.value = true
}

const onClose = () => {
  isOnline.value = false
}

const onMessage = (event) => {
  // 获取服务端推送过来的消息
  let dataString = event.data
  // 将 dataString 转换为 json 对象
  let response = JSON.parse(dataString)

  console.log('服务端推送过来的消息：', response)

  // 判断是否是系统消息
  if (response.system) {
    // 系统消息 好友列表展示
    let names = response.message
    friendsList.value = []
    systemMessages.value = []
    for (let i = 0; i < names.length; i++) {
      if (names[i] !== username.value) {
        friendsList.value.push(names[i])
        systemMessages.value.push(names[i])
      }
    }
  } else {
    // 非系统消息
    let history = sessionStorage.getItem(response.fromName);
    if (!history) {
      historyMessage.value = []
    }
    historyMessage.value.push(response)
    sessionStorage.setItem(response.fromName, JSON.stringify(historyMessage.value))
  }
}

const showChat = (name) => {
  toName.value = name

  // 清除聊天区的数据
  let history = sessionStorage.getItem(toName.value)
  if (!history) {
    historyMessage.value = []
  } else {
    historyMessage.value = JSON.parse(history)
  }

  // 展示聊天对话框
  isShowChat.value = true

  // 显示正在和谁聊天
  chatMes.value = true
}

const submit = () => {
  sendMessage.value.message = sendMessage.value.message.trim()

  if (!sendMessage.value.message) {
    ElMessage.error('请输入聊天内容')
    return
  }

  sendMessage.value.toName = toName.value

  historyMessage.value.push(JSON.parse(JSON.stringify(sendMessage.value)))
  sessionStorage.setItem(toName.value, JSON.stringify(historyMessage.value))

  webSocket.value.send(JSON.stringify(sendMessage.value));
  sendMessage.value.message = ''
}
</script>

<style scoped>
@import '@/css/chat.css';
@import '@/css/bootstrap.min.css';

.wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: linear-gradient(135deg, #667eea, #764ba2) repeat;
}

.guest-name {
  margin-left: 1%;
  font-size: 12px;
}

.hostname {
  font-size: 12px;
  margin-bottom: -2%;
  text-align: right;
}
</style>
