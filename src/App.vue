<template>
  <v-app v-cloak>
    <v-toolbar fixed app :clipped-left="true" primary color="primary">
      <v-avatar :size="40" v-if="hasChurchLogo()">
        <img :src="getChurchLogoSrc()" :alt="getChurchLogoAlt()">
      </v-avatar>
      <v-toolbar-title class="title--caps white--text">{{ getChurchName() }}{{$t('title')}}</v-toolbar-title>
      <v-spacer></v-spacer>
      <div>
        <i class="flag-icon flag-icon-de" v-bind:class="{ active: activeLang=='de'  }" @click="setDE"></i>
        <i class="flag-icon flag-icon-us" v-bind:class="{ active: activeLang=='en'  }" @click="setEN"></i>
      </div>
      <v-btn fab small dark color="primary" @click="showSettingsModal = true"><v-icon>settings</v-icon></v-btn>
    </v-toolbar>
    <v-content>
      <v-container fluid fill-height>
        <index-view/>
      </v-container>
    </v-content>
    <v-dialog v-model="showSettingsModal" max-width="500px">
      <settings-view @close="closeSettings"></settings-view>
    </v-dialog>
  </v-app>
</template>

<script>
import IndexView from './components/IndexView'
import SettingsView from './components/settings/SettingsView'
import Vue from 'vue'
import { mapState } from 'vuex';
import keys from './config/LocalForageKeys'
import localFileManager from './utilities/localFileManager'
const path = require("path");
const fs = require('fs');
const filePath = path.join(__dirname, "", "settings/settings.json");
const templateFilePath = path.join(__dirname, "", "template/template.json");
const ct = require('./churchtools/credentials').churchtools;

export default {
  name: 'App',
  components: {
    IndexView,
    SettingsView
  },
  data () {
    return {
      title: Vue.i18n.translate('title'),
      activeLang: 'de',
      showSettingsModal: false
    }
  },
  methods: {
    setEN: function() {
      Vue.i18n.set('en');
      this.activeLang = 'en'
    },    
    setDE: function() {
      Vue.i18n.set('de');
      this.activeLang = 'de'
    },
    loadData: function() {
      let rawdata
      let _settings
      let _template

      _settings = localFileManager.get(localStore, keys.SETTINGS)
      _template = localFileManager.get(localStore, keys.TEMPLATE)
      
      ct.host = _settings.churchtoolshost
      this.$store.dispatch('UPDATE_SETTINGS', _settings)
      this.$store.dispatch('UPDATE_TEMPLATE', _template)
    },
    closeSettings: function() {
      this.showSettingsModal = false
    },
    getChurchName: function() {
      if (this.settings.churchname) {
        return this.settings.churchname + " - "
      }
    },
    hasChurchLogo: function () {
      let hasLogo = false;
      if (this.settings && this.settings.churchlogo && this.settings.churchlogo.src) {
        hasLogo = true
      }
      return hasLogo
    },
    getChurchLogoSrc: function () {
      if (this.settings && this.settings.churchlogo) {
        return this.settings.churchlogo.src
      }
    },
    getChurchLogoAlt: function () {
      if (this.settings  && this.settings.churchlogo) {
        return this.settings.churchlogo.description
      }
    },
  },
  created(){
    this.loadData()
  },
  computed: mapState([
    'settings'
  ])
}
</script>

<style scoped>
  .languages {
    background-color:rgba(255, 255, 255, 0.10);
  }
  
  [v-cloak] {
    background-color: #CCC;
  }

  .active {
    opacity: 1;
  }
  .v-icon {
    display: inline-flex !important;
    line-height: 1 !important;
  }
</style>