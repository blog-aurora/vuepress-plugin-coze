<template>
  <!--这是一个登录注册的组件-->
  <div class="coze-custom-login">
    <slot name="cozeCustomTop"></slot>
    <form v-on:submit.prevent>
      <div class="coze-custom-item-common">
        <input autocomplete type="text" placeholder="请输入用户名" v-model="username" name="username">
      </div>
      <div class="coze-custom-item-common">
        <input autocomplete placeholder="输入6到20位包含数字,字母密码" v-model="password" name="password" type="password">
      </div>

      <div class="coze-custom-item-common">
        <input autocomplete placeholder="请输入邮箱" v-model="email" name="email" type="text">
      </div>

      <slot name="cozeCustomCenter"></slot>
      <div class="coze-custom-item-common coze-custom-button">
        <button @click="verifyIdentify">注册</button>
      </div>
      <div class="coze-custom-item-common coze-custom-button">
        <button @click="loginOut">登出</button>
      </div>
    </form>
    <slot name="cozeCustomBottom"></slot>
  </div>
</template>

<script>
const AV = require('leancloud-storage');
const { Query, User } = AV;
const appId = __APP_ID__;
const appKey = __APP_KEY__;
const masterKey = __Master_Key__;
const avatar = __AVATAR_PATH__;
export default {
  name: "CozeLogin",
  data() {
    return {
      password: '',
      username:'',
      email: '',
      verifyText: '请注册(●￣(ｴ)￣●)',
    }
  },
  emits: ['cozeLoginOut','cozeLogin'],
  methods: {
    loginOut() {
      const currentUser = AV.User.current();
      if (currentUser) {
        this.verifyText = '你已经退出登录(●￣(ｴ)￣●)'
        AV.User.logOut();
        this.$emit("cozeLoginOut",{
          status: 1,
          message: "成功退出登录"
        })
      } else {
        this.verifyText = '你未登录或者注册(●￣(ｴ)￣●)'
        this.$emit("cozeLoginOut",{
          status: 0,
          message: '未登录或者注册'
        })
      }
    },
    setMoodClass() {
      const Talk = AV.Object.extend('Talk');
      const talk = new Talk();

      let photoArr = [{
        objectId: "61869d9b49cd7e59a03e09c8",
        photoName: "1GyQsaZ8MhjTVKi.jpg",
        photoUrl: "https://ooszy.cco.vin/img/blog-note/image-20211014230618458.png"
      },
        {
          objectId: "61869d9b49cd7e59a03e09c8",
          photoName: "1GyQsaZ8MhjTVKi.jpg",
          photoUrl: "https://ooszy.cco.vin/img/blog-note/image-20211009083404086.png"
        }
      ]
      talk.set('mood_title', "Vuepress-theme-Aurora");
      talk.set('mood_content', "A vuepress-based animation blog theme, simple, beautiful, multi-color, multiple custom functions, " +
          "providing article poster sharing, talk, photo album, comment and other features," +
          "一款基于vuepress的动漫类博客主题，简洁，漂亮，多色彩，多种自定义功能，提供文章海报分享，说说，相册，评论，侧边栏，" +
          "自动生成文章侧边栏等特色功能\n" +
          "主题地址: https://github.com/qsyyke/vuepress-theme-aurora");
      talk.set("mood_like",1000)
      talk.set("mood_comment","")
      talk.set("mood_photos",photoArr)
      talk.set("mood_show",true)
      talk.set("mood_user",this.username)
      talk.save().then(() => {
        setTimeout(() => {
          this.verifyText = "设置成功,欢迎小主(●￣(ｴ)￣●)"
        },3000)
        return
      },(err) => {
        setTimeout(() => {
          this.verifyText = "设置失败,正在重新设置..."
        },3000)
        this.setMoodClass()
      })
    },
    isEmail(str){
      let reg = /^[a-z0-9]+([._\\-]*[a-z0-9])*@([a-z0-9]+[-a-z0-9]*[a-z0-9]+.){1,63}[a-z0-9]+$/g
      return reg.test(str)
    },
    isValidUsername(str){
      let reg = /^[0-9a-zA-Z|_]{4,20}$/g
      return reg.test(str)
    },
    isValidPassword(str){
      if(str.length<6||str.length>20){
        return false
      }
      if(/[^a-zA-Z0-9_]/.test(str)){
        return false
      }
      if(/(^[a-z]+$|^[A-Z]+$|^\d+$|^_+$)/.test(str)){
        return false
      }
      return true
    },
    verifyIdentify() {
      if (!this.isEmail(this.email) || !this.isValidUsername(this.username) || !this.isValidPassword(this.password)) {
        this.verifyText = "请正确输入信息(●￣(ｴ)￣●)"
        this.$emit("cozeLogin",{
          registerStatus: 0,
          message: '输入信息不符合要求',
          administrator: 0,
          error: '',
          username: this.username,
          email: this.email,
          password: this.password
        })
      }else {
        let administrator = 0
        //判断是否存在talk数据
        new Promise((resolve,reject) => {
          const query = new AV.Query('Talk');
          query.count().then((count) => {
            resolve()
          },(err) => {
            //没有数据，不存在这个TalkClass
            administrator = 1
            resolve()
          });
        }).then(() => {
          const user = new AV.User();
          user.setUsername(this.username);
          user.setPassword(this.password);
          user.setEmail(this.email);
          user.set('administrator', administrator);
          user.signUp().then((user) => {
            this.$emit("cozeLogin",{
              registerStatus: 1,
              message: '注册成功',
              administrator: administrator,
              error: '',
              username: this.username,
              email: this.email,
              password: this.password
            })
            // 注册成功
            if (administrator === 1) {
              this.verifyText = "注册成功,正在进行设置,请等待"
              this.setMoodClass()
            }else {
              this.verifyText = "注册成功,欢迎小主(●￣(ｴ)￣●)"
            }
          }, (error) => {
            this.verifyText = error
            // 注册失败（通常是因为用户名已被使用）
            this.$emit("cozeLogin",{
              registerStatus: 0,
              message: '注册失败',
              administrator: administrator,
              error: error,
              username: this.username,
              email: this.email,
              password: this.password
            })
          });
        })
      }
    },
  }
}
</script>