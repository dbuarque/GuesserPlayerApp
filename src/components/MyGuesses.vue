<template>
  <div class="justify-content-md-center margin">
    <MetamaskAlert v-if='showMetamask'/>

    <!--Alert-->
    <notifications group="copyAlert"
                   position="bottom right"
                   width="120"
                   :speed="500">
    <template slot="body" slot-scope="props">
      <div class="copyAlert">
        <div class="copyAlert-content">
          Url copied!
        </div>
      </div>
    </template>
    </notifications>


    <h2>Profile</h2>
    <br>
    <b-row align-g="start" style="margin: 0 !important">
      <b-col style="padding-left: 0">
        <b-row style="margin: 0 !important">
          <b-col cols="12" sm="6" md="3" lg="3" style="padding-left: 0; margin-bottom: 20px">
            <b-row class="justify-content-md-center">
              <span class="avatar">
              <qrcode :value="address" :options="{ foreground: color1, background: color2, size: 150 }"></qrcode>
              </span>
            </b-row>
          </b-col>
          <b-col align-self="center" style="padding-left: 0; margin-bottom: 20px">
            <b-row align-h="start" style="margin: 0 !important">
              <span>
                <big>Address:</big>
              <div class="address-holder" v-clipboard:copy="address" style="overflow:hidden; cursor: pointer; max-width: 80vw"><small>{{address}}</small></div>
              </span>
            </b-row>
          </b-col>
        </b-row>
      </b-col>
    </b-row>
    <br>
    <h2>Events I have participated in</h2>
    <br>
    <b-card no-body>
      <b-tabs v-model="tabIndex">
        <b-tab class="wrapper10" title="Current Events" :title-link-class="linkClass(0)">
          <CurrentGuesses/>
        </b-tab>
        <b-tab class="wrapper10" title="Events being validated" :title-link-class="linkClass(1)">
          <ValidatingGuesses/>
        </b-tab>
        <b-tab class="wrapper10" title="Past events" :title-link-class="linkClass(2)">
          <PastGuesses/>
        </b-tab>
        <b-tab class="wrapper10" title="Created events" :title-link-class="linkClass(3)" title-link-class="end">
          <CreatedGuesses/>
        </b-tab>
      </b-tabs>
    </b-card>

    <div v-if="chartsShow" class="charts">
      <Charts/>
    </div>
  </div>

</template>

<script>
// import NetworkHelper from '@/js/NetworkHelper'
import GuessHelper from '@/js/Guess'
import ServerHelper from '@/js/ServerHelper'

import CurrentGuesses from './MyGuesses/CurrentGuesses.vue'
import ValidatingGuesses from './MyGuesses/ValidatingGuesses.vue'
import PastGuesses from './MyGuesses/PastGuesses.vue'
import CreatedGuesses from './MyGuesses/CreatedGuesses.vue'
import Charts from './MyGuesses/Charts.vue'
import Qrcode from '@xkeshi/vue-qrcode'
import MetamaskAlert from './Common/MetamaskAlert.vue'

export default {
  name: 'MyGuesses',
  data () {
    return {
      address: '0x0000000000000...',
      username: '',
      tabIndex: 0,
      chartsShow: false,
      color1: 'white',
      color2: '#ff0d73',
      showMetamask: false
    }
  },
  components: {
    MetamaskAlert,
    CurrentGuesses,
    ValidatingGuesses,
    PastGuesses,
    CreatedGuesses,
    Charts,
    Qrcode
  },
  methods: {
    linkClass (idx) {
      if (this.tabIndex === idx) {
        return ['bg-primary', 'text-light']
      } else {
        return ['bg-light', 'text-info']
      }
    },
    getUserName () {
      ServerHelper.getUser(GuessHelper.address[0]).then((data) => {
        this.username = data.username
      }).catch((err) => {
        return err
      })
    }
  },
  created: function () {
    let self = this

    GuessHelper.init().then(() => {
      GuessHelper.getAddressRefreshed().then((addresses) => {
        self.address = addresses[0]
        if (addresses === null ||
            addresses.length === 0) {
          self.showMetamask = true
          self.address = addresses[0]
        } else {
          // Checking if the user is connected to the right network
          window.web3.eth.net.getId().then(netId => {
            switch (netId) {
              case 4:
                break
              default:
                self.showMetamask = true
            }
          })
        }
      })
    })
    /*
    NetworkHelper.init().then(() => {
      if (NetworkHelper.state === 'disconnected' ||
        NetworkHelper.state === 'locked') {
        this.$router.push('signup')
      } else {
        GuessHelper.init().then(() => {
          this.address = GuessHelper.address[0]
          this.getUserName()
        })
      }
    })
    */
  }
}
</script>

<style lang="scss">

$avatarfor: white;
$avatarback: #ff0d73;

.margin{
  margin: 0% 10%;
  padding: 2% 0%;
}
.wrapper10{
  padding: 15px 2%;
}
.charts{
  padding: 3% 7%;
}
.address-holder{
  color: gray;
  background-color: white;
  border-radius: 5px;
  box-shadow: inset  0 0 2px #000000;
  padding: 0.6rem 0.6rem;
  text-align: left;
}
.avatar {
  padding: 15px;
  padding-bottom: 8px;
  border-radius: 10px;
  background: $avatarback;
}
.end {
  position: inherit;
}
</style>
