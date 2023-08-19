<template>
  <v-container id="homeContainer" :class="`h-100 py-0 state-${connectionState.toLowerCase()}`">
    <!-- Public Server Hint -->
    <v-dialog v-if="!store.isShowStoreAd" :value="store.requestedPublicServerProfileId != null" width="500">
      <v-card>
        <v-card-title class="headline grey lighten-2" v-html="$t('publicServerWarningTitle')" />
        <v-card-text v-html="$t('publicServerWarning')" class="pt-4" />
        <v-card-text>
          <strong>{{ $t("warning") }}! {{ $t("privacyWarning") }}</strong>
          <br />
          <a href="https://www.vpnhood.com/privacy-policy" target="_blank">{{ $t("readPrivacyPolicy") }}</a>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="store.lastServerHintId = null; store.requestedPublicServerProfileId = null;"
            v-text="$t('cancel')" />
          <v-btn color="primary" text @click="store.isShowStoreAd = true"
            v-text="$t('accept')" />
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Store Ad -->
    <v-dialog v-model="store.isShowStoreAd">
      <div id="storeAd" class="py-5 px-7 rounded-lg">
<!--        <v-btn v-if="!isMaximizeStoreAd" id="closeAddBtn" icon @click="store.isShowStoreAd = false; store.lastServerHintId = null; store.requestedPublicServerProfileId = null;">
          <v-icon>mdi-window-close</v-icon>
        </v-btn>-->
        <img  src="../assets/images/add-icons.png" width="100%" alt="Store Ad icons" />
        <h3 id="adTitle" class="title-bold color-sharp-master-green text-uppercase pb-2 mb-5 mt-5">{{$t("storeAdTitle")}}</h3>
        <p id="adDesc" class="regular-font text--white text-left mb-7" v-html="$t('storeAdDescription')"></p>
        <a id="goPremiumBtn" :class="[isMaximizeStoreAd ? 'sharp-btn' : 'sharp-bordered-btn', 'body-2 text-capitalize py-3 mb-4 rounded-xl']" href="https://play.google.com/store/apps/details?id=com.vphood.store.android" target="_blank"
           v-text="$t('goPremiumWithVpnHoodStore')" ></a>
        <v-btn v-if="!isMaximizeStoreAd" id="continueBtn" block height="auto" rounded class="sharp-btn text-capitalize px-3 py-3" @click="store.connect(store.requestedPublicServerProfileId, true);store.isShowStoreAd = false;">
          {{$t('continueWithFreeSlowSpeed')}}<v-icon class="ml-2">mdi-arrow-right-thin</v-icon>
        </v-btn>
        <v-btn v-else color="primary" outlined block rounded class="text-capitalize px-3 py-5" @click="store.isShowStoreAd = false; isMaximizeStoreAd = false;">
          {{$t('close')}}
        </v-btn>
      </div>
    </v-dialog>

    <v-snackbar top :timeout="-1" :value="store.state.isWaitingForAd" class="body-2">
      <div>
        <p class="text-center subtitle-2"> You have connected to a<br><b class="yellow--text">Free Public VpnHood
            Server!</b>
        </p>
        <p class="text-justify"> VpnHood offers a free, open-source solution to bypass censorship. We need you to watch
          an ad to help us maintain these servers. Please wait... </p>
      </div>
    </v-snackbar>

    <!-- suppress-to -->
    <v-snackbar top :timeout="-1" class="body-2" color="deep-purple accent-4"
      @input="store.connectionHint.sessionSuppressedBy = true"
      :value="!store.connectionHint.sessionSuppressedBy && store.state.sessionStatus != null && store.state.sessionStatus.suppressedBy != 'None'">
      <div v-if="store.state.sessionStatus != null">
        <span class="text-justify">{{ $t("sessionSuppressedByOther") }}</span>
      </div>
      <template v-slot:action="{ attrs }">
        <v-btn text v-bind="attrs" @click="store.connectionHint.sessionSuppressedBy = true">
          <v-icon small>close</v-icon>
        </v-btn>
      </template>
    </v-snackbar>

    <!-- suppress-by -->
    <v-snackbar top :timeout="-1" class="body-2" color="deep-purple accent-4"
      @input="store.connectionHint.sessionSuppressedTo = true"
      :value="!store.connectionHint.sessionSuppressedTo && store.state.sessionStatus != null && store.state.sessionStatus.suppressedTo == 'Other'">
      <div v-if="store.state.sessionStatus != null">
        <span class="text-justify"> {{ $t("sessionSuppressedByOther") }}</span>
      </div>
      <template v-slot:action="{ attrs }">
        <v-btn text v-bind="attrs" @click="store.connectionHint.sessionSuppressedTo = true">
          <v-icon small>close</v-icon>
        </v-btn>
      </template>
    </v-snackbar>

    <!-- upgrade -->
    <v-snackbar top :timeout="-1" :class="'version-notify body-2' + 'version-' + store.state.versionStatus"
      vertical="vertical" :color="store.state.versionStatus == 'Deprecated' ? 'warning' : 'info'"
      @input="store.updateHintDate = new Date()"
      :value="!store.updateHintDate &&
        store.state.lastPublishInfo != null &&
        store.state.lastPublishInfo.installationPageUrl && (store.state.versionStatus == 'Old' || store.state.versionStatus == 'Deprecated')">
      <div>
        <span v-if="store.state.versionStatus == 'Deprecated'" class="text-justify">
          {{ $t("versionIsDeprecated") }}</span>
        <span v-if="store.state.versionStatus == 'Old'" class="text-justify"> {{ $t("versionIsOld") }}</span>
        <p></p>
        <div class="text-justify"> {{ $t("currentVersion") }} {{ store.formatVersion(store.appVersion()) }}</div>
        <div class="text-justify"> {{ $t("newVersion") }} {{ store.appNewVersion() }}</div>
      </div>
      <template v-slot:action="{ attrs }">
        <v-btn v-if="store.state.lastPublishInfo" text :href="store.state.lastPublishInfo.installationPageUrl"
          target="_blank">
          {{ $t("update") }}
        </v-btn>
        <v-btn text v-bind="attrs" @click="store.updateHintDate = new Date()">
          {{ $t("ignore") }}
        </v-btn>
      </template>
    </v-snackbar>

    <v-row class="align-center h-100">
      <!-- Minimize store ad -->
      <div id="minimizeStoreAd" v-if="store.isShowMinimizeStoreAd" @click="store.isShowStoreAd = true; isMaximizeStoreAd = true;">
        <img  src="../assets/images/add-icons-minimize.png" width="40px" alt="Store Ad icons" />
        <h3 id="minimizeAdTitle" class="title-bold color-sharp-master-green text-capitalize ml-2">{{$t("storeAdTitle")}}</h3>
      </div>
      <!-- Circle -->
      <v-col cols="12" sm="6" id="middleSection" class="text-center align-self-center">
        <div class="my-card-view">
          <!-- Speed -->
          <div id="speedSection" class="text-center d-inline-flex mb-8">
            <div class="mx-2">
              <span class="speedLabel">{{ $t("downloadSpeed") }}:</span>
              <span class="speedValue"> {{ this.formatSpeed(this.store.state.speed.received) }} </span>
              <span class="speedUnit">Mbps</span>
            </div>
            <div class="mx-2">
              <span class="speedLabel">{{ $t("uploadSpeed") }}:</span>
              <span class="speedValue"> {{ this.formatSpeed(this.store.state.speed.sent) }} </span>
              <span class="speedUnit">Mbps</span>
            </div>
          </div>
          <div id="circleOuter" class="">
            <div id="circle">
              <div id="circleContent" class="align-center">
                <span id="stateText">{{ store.connectionStateText("$") }}</span>
                <!-- usage -->
                <div v-if="connectionState == 'Connected' && this.bandwidthUsage()">
                  <div id="bandwidthUsage">
                    <span>{{ this.bandwidthUsage().used }} of</span>
                  </div>
                  <div id="bandwithTotal" v-if="connectionState == 'Connected'">
                    <span>{{ this.bandwidthUsage().total }}</span>
                  </div>
                </div>
                <!-- check -->
                <v-icon class="state-icon" v-if="stateIcon != null" size="50" color="white">{{ this.stateIcon }}
                </v-icon>
              </div>
            </div>
          </div>
        </div>
      </v-col>
      <!-- Connect buttons -->
      <v-col cols="12" order-sm="12" align-self="start" class="text-center">
        <!-- Connect Button -->
        <v-btn v-if="connectionState == 'None'" id="connectButton" class="main-button py-sm-8"
          @click="store.connect('$')"> {{ $t("connect") }} </v-btn>
        <!-- Diconnect Button -->
        <v-btn v-if="
          connectionState == 'Waiting' ||
          connectionState == 'Connecting' ||
          connectionState == 'Connected' ||
          connectionState == 'Diagnosing'
        " id="disconnectButton" class="main-button py-sm-8" @click="store.disconnect()">
          <span>{{ $t("disconnect") }}</span>
        </v-btn>
        <!-- Diconnecting -->
        <v-btn v-if="connectionState == 'Disconnecting'" id="disconnectingButton" class="main-button py-sm-8"
          style="pointer-events: none">
          <span>{{ $t("disconnecting") }}</span>
        </v-btn>
      </v-col>
      <!-- Config (Bottom btn)-->
      <v-col cols="12" sm="6" id="configSection" class="align-self-end align-self-sm-auto pb-0">
        <!--Card view apply style only on min-width 600px-->
        <div class="my-card-view">
          <!-- *** ipFilter *** -->
          <v-btn depressed block class="config-btn mb-2" @click="showIpFilterSheet()">
            <v-icon class="config-icon">public</v-icon>
            <span class="config-label">{{ $t("ipFilterStatus_title") }}</span>
            <v-icon class="config-arrow" flat>keyboard_arrow_right</v-icon>
            <span class="config">{{ this.clientCountryFilterStatus }}</span>
            <v-img v-if="!store.userSettings.tunnelClientCountry"
              :src="store.getIpGroupImageUrl(store.state.clientIpGroup)" max-width="24" class="ma-1" />
          </v-btn>
          <!-- *** appFilter *** -->
          <v-btn v-if="
            store.features.isExcludeAppsSupported ||
            store.features.isIncludeAppsSupported
          " depressed block class="config-btn mb-2" @click="showAppFilterSheet()">
            <v-icon class="config-icon">apps</v-icon>
            <span class="config-label">{{ $t("appFilterStatus_title") }}</span>
            <v-icon class="config-arrow">keyboard_arrow_right</v-icon>
            <span class="config">{{ this.appFilterStatus }}</span>
          </v-btn>
          <!-- *** Protocol *** -->
          <v-btn depressed block class="config-btn mb-2" @click="showProtocolSheet()">
            <v-icon class="config-icon">settings_ethernet</v-icon>
            <span class="config-label">{{ $t("protocol_title") }}</span>
            <v-icon class="config-arrow" flat>keyboard_arrow_right</v-icon>
            <span class="config">{{ protocolStatus }}</span>
          </v-btn>
          <!-- *** server *** -->
          <v-btn depressed block class="config-btn" @click="showServersSheet()">
            <v-icon class="config-icon">dns</v-icon>
            <span class="config-label">{{ $t("selectedServer") }}</span>
            <v-icon class="config-arrow" flat>keyboard_arrow_right</v-icon>
            <span class="config">{{ store.clientProfile.name("$") }}</span>
          </v-btn>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<style>
.v-input--checkbox .v-label {
  font-size: 12px;
}
</style>

<script>

export default {
  name: "HomePage",
  data(){
    return{
      isMaximizeStoreAd: false,
    }
  },
  created() {
    this.store.checkStoreAdStatus();

    this.store.setTitle(this.$t("home"));
    this.monitorId = setInterval(() => {
      if (!document.hidden)
        this.store.updateState();
    }, 1000);

  },
  beforeDestroy() {
    clearInterval(this.monitorId);
    this.monitorId = 0;
  },
  computed: {
    connectionState() { return this.store.connectionState("$"); },
    appFilterStatus() {
      let appFilters = this.store.userSettings.appFilters;
      if (!appFilters) appFilters = [];

      if (this.store.userSettings.appFiltersMode == 'Exclude') return this.$t("appFilterStatus_exclude", { x: appFilters.length });
      if (this.store.userSettings.appFiltersMode == 'Include') return this.$t("appFilterStatus_include", { x: appFilters.length });
      return this.$t("appFilterStatus_all");
    },
    ipFilterStatus() {
      let ipGroupFilters = this.store.userSettings.ipGroupFilters;
      if (!ipGroupFilters) ipGroupFilters = [];

      if (this.store.userSettings.ipGroupFiltersMode == 'Exclude') return this.$t("ipFilterStatus_exclude", { x: ipGroupFilters.length });
      if (this.store.userSettings.ipGroupFiltersMode == 'Include') return this.$t("ipFilterStatus_include", { x: ipGroupFilters.length });
      return this.$t("ipFilterStatus_all");
    },
    clientCountryFilterStatus() {
      return this.store.userSettings.tunnelClientCountry
        ? this.$t("ipFilterStatus_all")
        : this.$t("ipFilterStatus_excludeClientCountry");
    },
    protocolStatus() {
      return (this.store.userSettings.useUdpChannel) ? this.$t('protocol_udpOn') : this.$t('protocol_udpOff');
    },
    stateIcon() {
      if (this.connectionState == 'Connected' && !this.bandwidthUsage()) return "check";
      if (this.connectionState == 'None') return "power_off";
      if (this.connectionState == 'Connecting') return "power";
      if (this.connectionState == 'Diagnosing') return "network_check";
      if (this.connectionState == 'Waiting') return "hourglass_top";
      return null;
    }
  },
  methods: {
    async remove(clientProfileId) {
      clientProfileId = this.store.clientProfile.updateId(clientProfileId);
      const res = await this.$confirm(this.$t("confirmRemoveServer"), { title: this.$t("warning") })
      if (res) {
        await this.store.invoke("removeClientProfile", { clientProfileId });
        this.store.loadApp();
      }
    },

    editClientProfile(clientProfileId) {
      window.gtag('event', "editprofile");
      clientProfileId = this.store.clientProfile.updateId(clientProfileId);
      this.$router.push({ path: this.$route.path, query: { ... this.$route.query, editprofile: clientProfileId } })
    },

    showAddServerSheet() {
      window.gtag('event', "addServerButton");
      this.$router.push({ path: this.$route.path, query: { ... this.$route.query, addserver: '1' } })
    },

    showServersSheet() {
      window.gtag('event', "changeServer");
      this.$router.push({ path: this.$route.path, query: { ... this.$route.query, servers: '1' } })
    },

    showProtocolSheet() {
      window.gtag('event', "changeProtocol");
      this.$router.push({ path: this.$route.path, query: { ... this.$route.query, protocol: '1' } })
    },

    showAppFilterSheet() {
      window.gtag('event', "changeAppFilter");
      this.$router.push({ path: this.$route.path, query: { ... this.$route.query, appfilter: '1' } })
    },

    showIpFilterSheet() {
      window.gtag('event', "changeIpFilter");
      //this.$router.push({ path: this.$route.path, query: { ... this.$route.query, ipfilter: '1' } })
      this.$router.push({ path: this.$route.path, query: { ... this.$route.query, mycountry: '1' } })
    },

    bandwidthUsage() {

      if (!this.store.state || !this.store.state.sessionStatus || !this.store.state.sessionStatus.accessUsage)
        return null;

      let accessUsage = this.store.state.sessionStatus.accessUsage;
      if (accessUsage.maxTraffic == 0)
        return null;

      let mb = 1000000;
      let gb = 1000 * mb;
      var traffic = this.store.state.accountTraffic;

      let ret = { used: traffic.sent + traffic.received, total: accessUsage.maxTraffic };

      ret.total = ret.total >= gb ? (ret.total / gb).toFixed(1) + "GB" : (ret.total / mb).toFixed(0) + "MB";
      ret.used = ret.used >= gb ? (ret.used / gb).toFixed(1) + "GB" : (ret.used / mb).toFixed(0) + "MB";
      return ret;
    },

    formatSpeed(speed) {
      return (speed * 10 / 1000000).toFixed(2);
    },
  }
}
</script>
<style scoped>
#storeAd{
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  text-align: center;
  width: 100%;
  height: 100%;
  background-color: var(--dark-blue);
}
#closeAddBtn{
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: rgba(255, 255, 255, 0.05);
}
#closeAddBtn i{
  color: white;
  opacity: .5;
}
/*#addContent{
  border: 1px #16a4fd38 solid;
  padding: 30px 20px;
  border-radius: 25px;
}*/
#adTitle{
  font-size: 120%;
  border-bottom: 1px rgba(63, 246, 169, 0.15) solid;
}
#goPremiumBtn{
  text-decoration: none;
  display: block;
  width: 100%;
}
.sharp-btn{
  color: white;
  background-image: linear-gradient(to right, var(--sky-blue), var(--sharp-master-green));
  text-shadow: 0 0 4px #000000;
  box-shadow: 0 3px 5px -1px rgba(0, 0, 0, 0.2), 0 5px 8px 0 rgba(0, 0, 0, 0.14), 0 1px 14px 0 rgba(0, 0, 0, 0.12) !important;
}
.sharp-bordered-btn{
  color: var(--sharp-master-green);
  border: 1px var(--sharp-master-green) solid;
}
#adDesc{
  font-size: 17px;
  line-height: 31px;
}
#minimizeStoreAd{
  display: inline-flex;
  justify-content: center;
  align-items: center;
  border-radius: 50px;
  border: 1px var(--sharp-master-green) solid;
  padding:3px 15px;
  margin: 0 auto;
}
#minimizeAdTitle{
  font-size: 100%;
}
</style>
