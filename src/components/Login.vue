<template>
    <div id="pa_con" class="so-con warp page_login">
        <!--<header id="pa_head" class="login">
            <img src="static/frist/images/login_logo.png" alt="">
        </header>-->
        <div class="login_title" v-bind:style="{backgroundImage: 'url(' + logosrc + ')'}" >
          <!-- <img   src="static/frist/images/login_logo.png">              -->
        </div>  

        <div class="new_panel login_area">
            <div class="new_panel_top"></div>
            <div class="new_panel_center">
                <form>
                    <fieldset>
                        <div class="form_g account">
                            <legend>帐号</legend>
                            <input type="text" placeholder="请输入帐号" v-model="username" autocomplete="off"
                                   class="user-name" @input="checkUserName(username,'user-name')">
                            <i class="close close1" @click="ClearInput('close1','user-name')"></i>
                        </div>
                        <label class="error-message "></label>
                    </fieldset>
                    <fieldset>
                        <div class="form_g password">
                            <legend>密码</legend>
                            <input type="text" placeholder="请输入密码" v-model="password" ref='psw'
                                   onfocus="this.type='password'" autocomplete="off" class="pass-word"
                                   @input="checkpassword(password,'pass-word')">
                            <i class="close close2" @click="ClearInput('close2','pass-word')"></i>
                        </div>
                        <label class="error-message"> </label>
                    </fieldset>
                    <fieldset>
                        <div class="form_g password ">
                            <legend>验证码</legend>
                            <input type="text" placeholder="请输入验证码" autocomplete="off"  maxlength="4" v-model="yzmcode">
                            <img :src="verImgCode" alt="" @click="switchYzmcode()">
                        </div>
                        <label class="error-message "></label>
                    </fieldset>
                     <div class="no_login_box">
                         <input type="checkbox" id="longinCheck" v-model = 'checkStatu' ref='check'  name=""><label for="longinCheck">七天内免登陆</label>
                     </div>
                </form>
                <div class="btn btn_blue">
                    <a class="new_btn" href="javascript:;" @click="LoginAction()"><span class="big">登录</span></a>
                </div>
                <div class="other_link">
                    <a href="/reg">马上注册</a>
                    <a href="javascript:;" @click="demoPlay()">免费试玩</a>
                    <!-- <a href="javascript:;" @click="openGame('http://www.providesupport.com?messenger=0bxg1rx3vv8lc036lt4a265vdi')">在线客服</a> -->
                    <!--<a href="javascript:;" @click="openGameOnline()">在线客服</a>-->
                    <a :href="custUrl" target="_blank">在线客服</a>
                </div>
            </div>
        </div>
        <AutoCloseDialog ref="autoCloseDialog" text=" " type="" />
        <GoUrlDialog ref="goUrlDialog" text=" " type="" />
        
    </div>
</template>

<script>
// import $ from "jquery";
import Mixin from '@/Mixin'
import AutoCloseDialog from '@/components/publicTemplate/AutoCloseDialog'
import GoUrlDialog from '@/components/publicTemplate/GoUrlDialog'

export default {
  name: 'Login',
  mixins:[Mixin],
  components: {
    AutoCloseDialog,
    GoUrlDialog,
  },
    data: function() {
        return {
            username :'',
            password :'',
            verImgCode:'',
            yzmcode:'',
            client:'',
            submitflage: false ,
            custUrl:'',
            checkStatu:false,
            logosrc:'',
            errCheckCount : 0,
        }
    },
  created:function () {
      this.switchYzmcode()
  },
  mounted:function() {
       // this.username = 'admin' ;
      document.documentElement.scrollTop = document.body.scrollTop=0; // 回到顶部
      this.custUrl=localStorage.getItem('Url');
      this.getLoginIcon()
  },
  methods: {

      //清除model数据,cl元素class
      clearVal: function (cl) {
          if(cl=='pass-word'){
              this.password ='';}
          if(cl=='user-name'){
              this.username ='';
          }
      },
    //获取验证码；
    switchYzmcode:function () {
          let _self =this ;
          let url= this.action.uaa + 'apid/member/code/get?time='+ Math.random();
          $.ajax({
              type:"GET",
              url:url,
              success: (data) => {
                  _self.verImgCode = data.data && 'data:image/png;base64,' + data.data.code || '';
                  _self.client = data.data && data.data.clientId || '';
              }
          })
      },

    getLoginIcon:function(){
      var loginStr =  this.getCookie("siteData")
      var loginstrArray = JSON.parse(loginStr )
      this.logosrc = this.action.picurl+loginstrArray.h5LogoUrl+'/0'
      // console.log( this.logosrc ,'loginstrArray' )
      // console.log(loginstrArray)
    },
    // 登录接口 moved to 主页/index.vue
    LoginAction:function() {
          var _self = this ;
        console.log( _self.checkStatu )

        if(_self.submitflage){
            return false ;
        }
        if(this.username ==''){
            this.$refs.autoCloseDialog.open('请输入用户名') ;
            return false ;
        }
        if(this.password ==''){
            this.$refs.autoCloseDialog.open('请输入登录密码') ;
            return false ;
        }
        if(this.yzmcode==''){
            this.$refs.autoCloseDialog.open('请输入验证码') ;
            return false ;
        }
        var falg = $('.error-message').hasClass('red') ;  // 验证不通过，不允许提交
        if(falg){
            return false ;
        }
        _self.submitflage = true ;  
        var logindata ={}
        if(_self.checkStatu ){
              logindata = {  // grant_type: 'password', username: 'bcappid02|admin', password: 'admin'
              grant_type: 'password',
              username: 'bcappid02|'+this.username ,
              password: this.password ,
              code: this.yzmcode ,  // 验证码
              source: 2,
              tokenDay:7,
          }
        }else{
          logindata = {  // grant_type: 'password', username: 'bcappid02|admin', password: 'admin'
              grant_type: 'password',
              username: 'bcappid02|'+this.username ,
              password: this.password ,
              code: this.yzmcode ,  // 验证码
              source: 2,
          }
        }  
        $.ajax({
            type: 'post',
            headers: {clientId:this.client,Authorization: 'Basic d2ViX2FwcDo='},
           // url: this.action.uaa + 'oauth/token',
            url: this.action.uaa + 'apid/member/login',
            data: logindata ,
            success: (res) => {
               
                if(res.err == 'SUCCESS'){ // 登录成功
                    _self.submitflage = false ;
                    var cookieTime = 1*24*60*60*1000;
                    if(_self.checkStatu) {
                      cookieTime = 7*24*60*60*1000;
                    }
                    this.setCookie("access_token", res.data.access_token, cookieTime);  // 把登录token放在cookie里面
                    // this.setCookie("username", this.username);  // 把登录用户名放在cookie里面
                    this.setCookie("username", res.data.username,cookieTime);  // 把登录用户名放在cookie里面
                    // this.setCookie("psw", res.data.username,14);  // 把登录密码放在cookie里面

                    console.log( _self.username ,'username')
                    console.log( _self.password ,'password',)
                    this.setCookie("password", _self.password,cookieTime);
                    this.setCookie("uname", res.data.username,cookieTime);  //免登陆用
                    this.setCookie('acType',res.data.acType,cookieTime);   //把玩家类型放在cookie里面
                    this.$refs.autoCloseDialog.open('登录成功','','icon_check','d_check') ;
                      setTimeout(function () {
                        // console.log(logindata,'logindata' )
                          window.location = '/';
                          // _self.$router.push('');
                       },300)
                }else{
                    _self.submitflage = false ;
                    if ( res.code == 103 || res.code == 102 ) { // 检查用户名与密码，若用户名不存在(102)或用户名与密码不匹配(103)反馈“用户名或密码错误”
                        res.cnMsg = '用户名或密码错误';
                        if ( res.code == 103 ) {
                            this.errCheckCount++;
                        }
                    }
                    if ( this.errCheckCount > 2 && res.code == 103 ) {
                        this.$refs.goUrlDialog.open('用户名或密码错误，建议您联系客服', '在线客服', this.custUrl) ;
                    } else {
                        this.$refs.autoCloseDialog.open(res.cnMsg) ;
                        this.switchYzmcode()
                    }
                }

               this.$nextTick(function () {

               })
            },
            error: function () {
                _self.submitflage = false ;
            }
        });
    },
  }

}
</script>
<style type="text/css">
  /*.no_login_box{
    margin-left: -76px;
    margin-bottom: 5px;
  }*/

</style>
