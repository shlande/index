<template>
  <v-row class="pa-0">
    <!--侧边栏-->
    <v-col cols="3" class="pt-0">
      <!--大致信息-->
      <v-card flat>
        <div>
          <div class="avatar pt-6 pb-4 d-flex justify-center">
            <v-avatar
                size="64"
            >
              <v-img
                  :src="avatarUrl"
              >
              </v-img>
            </v-avatar>
          </div>
          <div class="uname pb-6 text-center">
            {{ name }}
          </div>
        </div>
      </v-card>
      <!--侧边栏-->
      <v-card class="mt-4 pa-0" flat>
        <v-list
            nav
        >
          <v-list-item-group color="primary" v-model="active">
            <v-list-item
                :value="0"
            >
              <v-list-item-icon>
                <v-icon>mdi-account</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>用户信息</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-list-item
                :value="1"
            >
              <v-list-item-icon>
                <v-icon>mdi-account</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>更改密码</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-card>
    </v-col>
    <!--信息展示-->
    <v-col cols="9" class="pa-0">
      <v-card height="100%" width="100%" flat class="pa-4">
        <!--用户信息-->
        <div v-if="active==0">
          <!--头像展示-->
          <div class="d-flex justify-center">
            <v-avatar
                size="128"
            >
              <v-img
                  :src="avatarUrl"
              >
                <v-sheet height="100%" width="100%" class="d-flex justify-center align-center"
                         style="background-color: rgba(0,0,0,.5)">
                  <div class="white--text">
                    修改头像
                  </div>
                </v-sheet>
              </v-img>
            </v-avatar>
          </div>
          <div class="d-flex justify-center">
            <v-card width="600" flat class="pt-4">
              <v-row>
                <v-col cols="12" class="pa-0">
                  <v-textarea
                      filled
                      name="description"
                      rounded
                      label="简介"
                      rows="3"
                      :placeholder="description"
                      v-model="info.description"
                  ></v-textarea>
                </v-col>
                <v-col cols="12" class="d-flex align-center">
                  <div style="width: 80px" class="text-end pr-4">昵称：</div>
                  <v-text-field
                      append-outer-icon="mdi-account"
                      :placeholder="name"
                      v-model="info.name"
                  >
                  </v-text-field>
                </v-col>
              </v-row>
              <div class="d-flex justify-end pt-8 pb-4">
                <v-btn color="primary" @click="setInfo">save</v-btn>
              </div>
            </v-card>
          </div>
        </div>
        <!--密码修改-->
        <div class="d-flex justify-center" v-if="active==1">
          <v-card width="600" class="pt-6" flat>
            <div class="d-flex align-center">
              <div style="width: 80px;" class="mr-2">当前密码：</div>
              <v-text-field
                  :type="pwd.show1?'text':'password'"
                  v-model="pwd.currentPWD"
                  :append-icon="pwd.show1?'mdi-eye-off':'mdi-eye'"
                  @click:append="pwd.show1=!pwd.show1"
              ></v-text-field>
            </div>
            <div class="d-flex align-center">
              <div style="width: 80px;" class="mr-2">新密码：</div>
              <v-text-field
                  :type="pwd.show2?'text':'password'"
                  v-model="pwd.newPWD"
                  :append-icon="pwd.show2?'mdi-eye-off':'mdi-eye'"
                  @click:append="pwd.show2=!pwd.show2"
              ></v-text-field>
            </div>
            <div class="d-flex align-center">
              <div style="width: 80px;" class="mr-2">重复：</div>
              <v-text-field
                  :type="pwd.show3?'text':'password'"
                  v-model="pwd.confirmPWD"
                  :append-icon="pwd.show3?'mdi-eye-off':'mdi-eye'"
                  @click:append="pwd.show3=!pwd.show3"
              ></v-text-field>
            </div>
            <div class="d-flex justify-end pt-8 pr-4">
              <v-btn color="primary" @click="this.setPassword" elevation="0">确认</v-btn>
            </div>
          </v-card>
        </div>
      </v-card>
    </v-col>
    <v-dialog
        v-model="dialog"
        persistent
        width="300"
    >
      <v-card
          color="primary"
          dark
      >
        <v-card-title>
          {{ dialogMsg.title }}
        </v-card-title>
        <v-card-text>
          {{ dialogMsg.content }}
        </v-card-text>
        <v-card-actions>
          <v-btn @click="dialog=false">确定</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>
<script>
import is_dev_env from "@/utils/is_dev_env";
import mock from "@/mock/user-full.json"
import {GetInfoRequest,EditInfoRequest } from "@/utils/proto/user/user_pb"
import {DownloadRequest} from "@/utils/proto/file/file.v2_pb"
export default {
  name: "User",
  props: {input: {}},
  created() {
    // if (is_dev_env()) {
    //   this.adapter(mock)
    //   return
    // }
    // this.adapter(this.input)
    this.getInfo()
  },
  computed: {
    avatarUrl() {
      if (this.avatar === undefined || this.avatar === "") {
        return "/avatar.png"
      }
      return this.avatar
    }
  },
  data() {
    return {
      dialogMsg: {
        title: "",
        content: "",
      },
      dialog: false,
      active: 0,
      name: "",
      avatar: "",
      email: "",
      description: "",
      info: {
        description: "",
        name: "",
        email: "",
      },
      pwd: {
        currentPWD: "",
        newPWD: "",
        confirmPWD: "",
        show1: false,
        show2: false,
        show3: false,
      }
    }
  },
  methods: {
    adapter(user) {
      this.name = user.name
      this.avatar = user.avatar
      this.description = user.description
      this.email = user.email
    },
    msgBox(title, message) {
      this.dialogMsg.title = title
      this.dialogMsg.content = message
      this.dialog = true
    },
    getInfo() {
      const req = new GetInfoRequest()
      // TODO: 需要修改
      // TODO: 身份绑定
      this.$client.userInfoPage(req,{authority:this.$store.state.token}).then(res => {
        this.name = res.getPublic().getNickname()
        this.avatar = this.getFile(res.getPublic().getAvatar())
        this.description = res.getPublic().getProfile()
        console.log(res)
      })
    },
    getFile(fid) {
      const req = new DownloadRequest()
      req.setFid(fid)
      console.log(fid)
      this.$client.download(req,{}).then(res => res.getUrl()).catch(err => console.error(err))
    },
    setPassword() {
      const old = this.pwd.currentPWD
      const pwd = this.pwd.newPWD
      if (this.pwd.confirmPWD !== this.pwd.newPWD) {
        this.msgBox("错误", "两次输入的新密码不相同")
        return
      }
      this.msgBox("错误", "暂时不支持修改密码")
    },
    setInfo() {
      if (is_dev_env()) {
        console.log(this.info,this.$store.state.token)
      }
      const req = new EditInfoRequest()
      req.setProfile(this.info.description)
      req.setNickname(this.info.name)
      // TODO: 身份绑定
      this.$client.editInfo(req,{authority:this.$store.state.token}).then(() => {
        this.msgBox("成功", "修改用户信息成功")
      }).catch(err => {
        this.msgBox("错误", err.message)
      })
    },
  }
}
</script>
