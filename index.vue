<template>
  <div id="appHardwareBasic" class="clearfix">
    <box-page>
      <div slot="header">
        <navigator :data="dataMenu"></navigator>
        <div class="col-xs-12 noPadding">
          <div class="col-xs-10 noPadding"></div>
          <div  :class="disabledDownload ? 'download-disabled' : 'download-report'" style="float:right;" @click="dowloadClick" >
          <div class="label-download">Download report:</div>
          <img src="../../assets/csv.svg" height="25" class="csv-icon noPadding " v-if="!disabledDownload">
        <img v-if="disabledDownload" height="25"  class="csv-icon" src="/icons/loader.svg">
          </div>
        </div>
      </div>
      <div class="page col-xs-12" slot="body">
        <div class="cont clearfix flex-height-section"> <!--167-->
          <router-view :user="user" :downloadEvasi="downloadEvasi" @evasi="click" @disabled="disabledEvasiDownload"
                       @no-disabled="noDisabledEvasiDownload"></router-view>
        </div>
      </div>
    </box-page>
  </div>
</template>

<script>
import {mapGetters} from 'vuex'
import BoxPage from '../global/template/BoxPage'
import Navigator from '../global/Navigator'

export default {
  name: 'hardwareBasic_index',
  components: {BoxPage, Navigator},
  watch: {},
  computed: {
    ...mapGetters({
      user: 'user/getUserInfo'
    }),
    dataMenu: function () {
      return [
        {
          title: 'Asset evasi',
          path: 'Purchase',
          active: true,
          demo: false
        },
        {
          title: 'Asset in evasione',
          path: 'Purchase in evasione',
          active: false,
          demo: false
        }]
    }
  },
  data () {
    return {
      noAuthMPS: false,
      downloadEvasi: false,
      loadingDownload: false,
      disabledDownload: false
    }
  },
  mounted () {
    this.downloadEvasi = false
  },
  methods: {
    disabledEvasiDownload: function () {
      this.disabledDownload = true
    },
    noDisabledEvasiDownload: function () {
      this.disabledDownload = false
    },
    click: function () {
      this.downloadEvasi = false
    },
    dowloadClick: function () {
      if(this.disabledDownload === false) {
        this.downloadEvasi = true
      } else {
        this.downloadEvasi = false
      }
    }
  }
}
</script>

<style lang="less" scoped>
@import "../../style/less/variabili.less";
@import "../../style/less/font.less";

#appHardwareBasic {
  height: 100%; //per RENT
  padding: 0px 10px;

  .download-report {
    cursor: pointer;
    margin-top: -75px;
    .set-font-bold(@size: 12px; @color: @myGray);
  }
.label-download{
  float: left;
  margin-right: 10px
}
  .download-disabled {
    cursor: not-allowed;
    margin-top: -75px;
    opacity: 0.3;
    pointer-events: all !important;
    .set-font-bold(@size: 12px; @color: @myGray);
  }

  .csv-icon {
    margin-top: -5px;
  }

  .page {
    height: 100%; //per RENT
    padding: 2px 5px 0px 0px;

    .cont {
      height: 100%; //per RENT
      overflow-y: auto;

      &::-webkit-scrollbar {
        width: 8px;
        height: 8px;
      }

      &::-webkit-scrollbar-track {
        border-radius: 3px;
        background: #e1e1e1;
      }

      &::-webkit-scrollbar-thumb {
        border-radius: 3px;
        background: #999;
      }
    }
  }
}

</style>
