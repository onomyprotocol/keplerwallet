<template>
  <section class="hero is-fullheight">
    <div class="hero-body">
      <div class="container">
        <div class="columns is-mobile is-centered">
          <div class="column is-6" >
              <h2 class="title" style="margin-top:24px; margin-left:5px;font-size:1.75rem" >{{ $t("msg.title") }}
              <span class="is-pulled-right" style="font-size:0.8rem;">v{{version}}</span>
              </h2>
          </div>
        </div>

        <div class="columns is-mobile is-centered">
          <div class="column"  v-bind:class="{'is-8': firstTime, 'is-6': !firstTime }">
            <div class="message is-primary is-small" v-show="firstTime" >
              <div class="message-header">
                <p>{{ $t("msg.welcome") }}</p>
                <button class="delete" aria-label="delete" @click="firstTime=false"></button>
              </div>
              <div class="message-body">
                <p>{{ $t("msg.login.walletExist") }}</p>
              </div>
            </div>
            <form class="box">
              <div class="field">
                <label class="label">{{ $t("msg.password") }}</label>
                <div class="control">
                  <input class="input" type="password" placeholder="********" required
                      :class="{'is-primary': error}" v-model="password">
                  </div>
                  <p class="help is-primary" v-if="error">{{ $t("msg.wrongPassword") }}</p>
                </div>
            
                <div class="field">
                  <button class="button is-link" @click.prevent="tryLogin" :class="{'is-loading': logining }">
                    {{ $t("msg.login_") }}
                  </button>
                </div>
            </form>
            <a class="button is-small is-text is-pulled-right" @click="openRemove=true">{{ $t("msg.remove.title") }}</a>
            <a class="button is-small is-text is-pulled-right" @click="openGnodeConfig=true">{{ $t("msg.gnodeConfigModal.title") }}</a>
          </div>
        </div>
        <remove :showModal="openRemove"></remove>
        <gnode-config-modal :showModal="openGnodeConfig"></gnode-config-modal>
      </div>
    </div>
  </section>
</template>

<script>
import { messageBus } from '@/messagebus'
import {isFirstTime} from '../../shared/first'
import Remove from '@/components/Remove'
import GnodeConfigModal from '@/components/GnodeConfigModal'

import {version, keplerNode, gnodeOption, keplerNode2, keplerLocalNode, platform} from '../../shared/config'

export default {
  name: "login",
    components: {
      Remove,
      GnodeConfigModal
  },
  data() {
    return {
      firstTime:false,
      password: '', 
      error: false,    
      openRemove: false,
      openGnodeConfig:false,
      version: version,
      logining: false
    }
  },
  created(){
    messageBus.$on('closeWindowRemove',()=>{this.openRemove = false})
    messageBus.$on('closeWindowGnodeConfig',()=>{this.openGnodeConfig = false})
  },
  mounted(){
    this.$log.info('isfirst(login.vue)? '+isFirstTime())
    this.firstTime = isFirstTime()
  },
  methods: {
    tryLogin(){
      if(this.logining)return
      this.logining = true

      let setPassword = this.$walletService.setPassword
      let password = this.password

      this.resetErrors()
      this.$walletService.initClient()
      let selectGnodeAndLogin = async function(){
        setTimeout(()=>this.checkLogin(), 2500)

        let localHeight
        let remoteHeight
        this.$log.debug('Time to select gnode.')
        this.$log.debug('Use local gnode? ' + gnodeOption.useLocalGnode)  
        this.$log.debug('Connect method is ' + gnodeOption.connectMethod)  
        if(!gnodeOption.useLocalGnode){
          this.$dbService.setGnodeLocation('remote')
          this.$log.debug('use remote kepler node.')
          return this.$walletService.startOwnerApi(this.password, keplerNode)
        }  
        if(gnodeOption.connectMethod ==='localAllTime'){
          this.$dbService.setGnodeLocation('local')
          this.$log.debug('use local kepler node.')
          return this.$walletService.startOwnerApi(this.password, keplerLocalNode)
        }else{
          this.$remoteGnodeService.getStatus().then(
            (res)=>{
              remoteHeight = parseInt(res.data.tip.height)
              this.$log.debug('Remote node height is ' + remoteHeight)
              if(gnodeOption.connectMethod ==='remoteAllTime' || gnodeOption.connectMethod ==='remoteFirst'){
                this.$dbService.setGnodeLocation('remote')
                this.$log.debug('use remote kepler node.')
                return this.$walletService.startOwnerApi(this.password, keplerNode)
              }
              this.$gnodeService.getStatus().then(
                (res)=>{
                  localHeight = parseInt(res.data.tip.height)
                  let peersCount = parseInt(res.data.connections)
                  this.$log.debug('local node height is ' + localHeight)
                  this.$log.debug('local node peers count is ' + peersCount)
                  if(localHeight + 60 >= remoteHeight && peersCount >= 3){
                    this.$log.debug('use local kepler node.')
                    this.$dbService.setGnodeLocation('local')
                    return this.$walletService.startOwnerApi(this.password, keplerLocalNode)
                  }else{
                    this.$log.debug('local node height is too low or peers too small. use remote kepler node.')
                    this.$dbService.setGnodeLocation('remote')
                    return this.$walletService.startOwnerApi(this.password, keplerNode)
                  }
              }).catch((err)=>{
                this.$log.error('local gnode failed. Use remote kepler node: ' + err)
                this.$dbService.setGnodeLocation('remote')
                return this.$walletService.startOwnerApi(this.password, keplerNode)
              })
            }
          ).catch((err)=>{
            this.$dbService.setGnodeLocation('local')
            this.$log.debug('use local kepler node.')
            return this.$walletService.startOwnerApi(this.password, keplerLocalNode)
          })
        }
      }
      setTimeout(()=>selectGnodeAndLogin.call(this), 800)
      this.resetErrors()
      },
    resetErrors(){
      this.error = false;
    },
    checkLogin(){
      this.$log.debug('check owner api process after try to login')
      this.$walletService.getAccounts().then(
        (res) =>{
          //this.$log.debug('getAccounts return:' + JSON.stringify(res.data))
          if(res.data.result.Ok){
            this.$log.debug('owner api process started')
            this.$walletService.setPassword(this.password)
            messageBus.$emit('logined')
          }else{
            return this.error = true
          }
        }).catch((error) => {
          this.$log.error('check owner api process got error:', error)
          return this.error = true
      }).finally(()=>{
            this.logining = false
      })
    }
  }
}
</script>

<style>
</style>
