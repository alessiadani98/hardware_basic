<template>
  <div id="asset-evasi">
    <div slot="body" class="chart-wrapper flex-height-section">
      <img v-if="ElmecTableLoading" style="height: 40px" src="/icons/loader.svg">
      <elmec-table
          v-if="!ElmecTableLoading"
          class="use-remaining-height-space"
          :data="ElmecTableValueData"
          :loading="ElmecTableLoading"
          @row-click="rowClicked"
          height="100%"
          size="mini"
      >
        <elmec-table-column :label="$t('Tipo')" prop="descrizioneFamiglia" :width="'120'">
          <template v-slot:header="scope">
          <elmec-table-filter
              :column-scope="scope"
              searchable
              :placement="'bottom-start'"
              :options="typeOptions"
              :type="'choice'"
              :optionDisplayKey="'descrizioneFamiglia'"
              :optionValueKey="'codiceFamiglia'"
              :value="ElmecTableMoreParams.famiglia"
              @filterChange="filterChange"
          ></elmec-table-filter>
          </template>
        </elmec-table-column>
        <elmec-table-column :label=" $t('Modello') " prop="descrizioneParte">
          <template v-slot:header="scope">
            <elmec-table-filter
                :column-scope="scope"
                :placement="'bottom-start'"
                :type="'text'"
                :value="ElmecTableMoreParams.descrizione_parte"
                @filterChange="filterChange"
            ></elmec-table-filter>
          </template>
        </elmec-table-column>
        <elmec-table-column :label=" $t('Brand') " prop="descrizioneBrand" :width="'115'">
          <template v-slot:header="scope">
            <elmec-table-filter
                :column-scope="scope"
                :placement="'bottom-start'"
                :options="brandOptions"
                searchable
                :type="'choice'"
                :optionDisplayKey="'descrizioneBrand'"
                :optionValueKey="'codiceBrand'"
                :value="ElmecTableMoreParams.brand"
                @filterChange="filterChange"
            ></elmec-table-filter>
          </template>
        </elmec-table-column>
        <elmec-table-column :label=" $t('NumeroSeriale') " prop="numeroDiSerie" :width="'150'">
          <template slot-scope="scope">
            <div v-if="scope.row.numeroDiSerie">
              {{ scope.row.numeroDiSerie }}
            </div>
            <div v-else> -</div>
          </template>
          <template v-slot:header="scope">
            <elmec-table-filter
                :column-scope="scope"
                :placement="'bottom-start'"
                :type="'text'"
                :value="ElmecTableMoreParams.serial_number"
                @filterChange="filterChange"
            ></elmec-table-filter>
          </template>
        </elmec-table-column>
        <elmec-table-column :label=" $t('DataEvasione') " prop="dataBolla" :width="'150'" >
          <template slot-scope="scope">
            {{ setFormatDate(scope.row.dataBolla) }}
          </template>
          <template v-slot:header="scope">
            <elmec-table-filter
                :placement="'bottom-start'"
                :column-scope="scope"
                dateValueFormat="yyyy-MM-dd"
                type="daterange"
                dateFromKey="datacons_da"
                dateToKey="datacons_a"
                :value="ElmecTableMoreParams.datacons_da || ElmecTableMoreParams.datacons_a ?
                [ElmecTableMoreParams.datacons_da ? ElmecTableMoreParams.datacons_da : '',
                 ElmecTableMoreParams.datacons_a ? ElmecTableMoreParams.datacons_a : ''] : []"
                @filterChange="dateFilterChangedHandler"
            >
            </elmec-table-filter>
          </template>
        </elmec-table-column>
        <elmec-table-column :label=" $t('PO') " prop="numeroPO" :width="'180'">
          <template slot-scope="scope">
            <div v-if="scope.row.numeroPO">
              {{ scope.row.numeroPO }}
            </div>
            <div v-else> -</div>
          </template>
          <template v-slot:header="scope">
            <elmec-table-filter
                :column-scope="scope"
                :placement="'bottom-start'"
                :type="'text'"
                :value="ElmecTableMoreParams.rif_po"
                @filterChange="filterChange"
            ></elmec-table-filter>
          </template>
        </elmec-table-column>
        <elmec-table-column :label=" $t('Bolla')" prop="numeroBolla" :width="'170'">
          <template slot-scope="scope">
            <div>
              <div style="float: left; min-width: 80px"> {{ scope.row.numeroBolla }}</div>
              <div class="link-download" style="float: left" @click.stop="downloadPDF(scope.row)"
                   v-if="scope.row.numeroBolla">
                <icon name="elm-download" class="elm-download"></icon>
              </div>
            </div>
          </template>
        </elmec-table-column>
      </elmec-table>
      <elmec-table-pagination
          v-if="ElmecTablePaginationData"
          :page-size="ElmecTablePaginationData.per_page"
          :total="ElmecTablePaginationData.total"
          :current-page="ElmecTablePaginationData.current_page"
          layout="total, ->, prev, pager, next"
          @current-change="ElmecTableSetPage"
      ></elmec-table-pagination>
    </div>
    <elmec-modal :visible=modalDetail @close="closeModalDetailAsset()" :title="selectedRow.descrizioneParte">
      <template v-slot:content>
      <span>
      <div class="col-xs-12 noPadding detail">
        <div class="col-xs-4 noPadding">{{ $t('Brand') }}: </div>
        <div class="col-xs-8 noPadding description">{{ selectedRow.descrizioneBrand }}</div>
      </div>
      <div class="col-xs-12 noPadding detail">
        <div class="col-xs-4 noPadding">{{ $t('NumeroSeriale') }}: </div>
        <div class="col-xs-8 noPadding description" v-if="selectedRow.numeroDiSerie">{{
            selectedRow.numeroDiSerie
          }}</div>
        <div v-else> - </div>
      </div>
      <div class="col-xs-12 noPadding detail">
        <div class="col-xs-4 noPadding">{{ $t('Garanzia') }}: </div>
        <div class="col-xs-8 noPadding description">{{ selectedRow.descrizioneGaranzia }}</div>
      </div>
      <div class="col-xs-12 noPadding detail">
        <div class="col-xs-4 noPadding">{{ $t('IndirizzoSpedizione') }}: </div>
        <div class="col-xs-8 noPadding description">
          <div class="col-xs-12 noPadding">{{ selectedRow.indirizzoUbicazione }}</div>
          <div class="col-xs-12 noPadding"> {{ selectedRow.capUbicazione }}</div>
          <div class="col-xs-12 noPadding"> {{ selectedRow.cittaUbicazione }}</div>
          <div class="col-xs-12 noPadding"> {{ selectedRow.provinciaUbicazione }}</div>
        </div>
      </div>
      <div class="col-xs-12 noPadding detail">
        <div class="col-xs-4 noPadding">{{ $t('DataConsegna') }}: </div>
        <div class="col-xs-8 noPadding description">{{ setFormatDate(selectedRow.dataBolla) }}</div>
      </div>
      <div class="col-xs-12 noPadding detail">
        <div class="col-xs-4 noPadding">{{ $t('PO') }}: </div>
        <div class="col-xs-8 noPadding description">{{ selectedRow.numeroPO }}</div>
      </div>
        <div class="col-xs-12 noPadding detail">
        <div class="col-xs-4 noPadding">{{ $t('Bolla') }}: </div>
        <div class="col-xs-8 noPadding description">
          {{ selectedRow.numeroBolla }}
          <div class="link-download" style="float: right; margin-right: 71%" @click="downloadPDF(selectedRow)">
                  <icon name="elm-download" class="elm-download"></icon>
                </div></div>
      </div>
        </span>
        <div class="col-md-12 separator"></div>
        <span>
        <div class="col-xs-12 noPadding title-ticket"> Ticket aperti:</div>
          <img v-if="isLoadingTicket" style="height: 40px; margin-left: 50%" src="/icons/loader.svg">
          <div v-else>
          <div v-if="ticketsPreview.length>0">
            <elmec-table
                ref="vuetable"
                :data="ticketsPreview"
                :loading="ElmecTableLoading"
                size="mini"
                height="100%"
                @row-click="goToTicket">
              <elmec-table-column label="ID" prop="ID"></elmec-table-column>
              <elmec-table-column label="Stato" prop="StatusID">
                <template slot-scope="scope">
                  <div class="status col-xs-12 col-md-1 noPadding" v-tooltip="getDescStatus(scope.row.StatusID)">
                    <elmec-icon v-if="getDescStatus(scope.row.StatusID) === 'Closed'" height="20"
                                :icon="'inTarget'"></elmec-icon>
                    <img v-else :src="'/icons/support/status/wip.svg'" height="20"/></div>
                </template>
              </elmec-table-column>
                 <elmec-table-column label="Apertura" prop="CreationDate"><template slot-scope="scope">
                {{ fmtCreation(scope.row.CreationDate) }}
              </template>
                 </elmec-table-column>
                  <elmec-table-column label="Ultimo aggiornamento" prop="LastUpdate" :width="'150'"><template
                      slot-scope="scope">
                {{ fmtLastUpdate(scope.row.LastUpdate) }}
              </template>
                 </elmec-table-column>
                <elmec-table-column label="Oggetto" prop="Subject" :width="'310'"></elmec-table-column>
            </elmec-table>
                <elmec-table-pagination
                    v-if="responseModal"
                    :page-size="responseModal.per_page"
                    :total="responseModal.total"
                    :current-page="responseModal.current_page"
                    layout="total, ->, prev, pager, next"
                    @current-change="ModalSetPage"
                ></elmec-table-pagination>
            </div>
          <div v-else class="no-ticket">Nessun ticket presente</div>
            </div>
      </span>
      </template>
    </elmec-modal>
  </div>
</template>

<script>
import {
  ElmecTable,
  ElmecTableColumn,
  ElmecTablePagination,
  ElmecTablePaginationMixin,
  ElmecIcon,
  ElmecTableFilter,
  ElmecModal
} from '@elmecglobal/elmec-ui'
import moment from 'moment'

export default {
  name: 'AssetEvasi',
  components: {
    ElmecTable,
    ElmecTableColumn,
    ElmecIcon,
    ElmecTablePagination,
    ElmecModal,
    ElmecTableFilter
  },
  mixins: [ElmecTablePaginationMixin],
  props: {
    downloadEvasi: {}
  },
  data () {
    return {
      ElmecTableUrl: process.env.VUE_APP_PURCHASE + '/mye/assetevasi?codicli=' + this.getUser().codiCli,
      ElmecTableDefaultParams: {
        page: 1,
        per_page: 20
      },
      ElmecTableUrlFilter: true,
      listaStatiComplete: [],
      brandOptions: [],
      typeOptions: [],
      listaTotal: [],
      responseModal: {},
      modalParams: {
        page: 1,
        per_page: 3
      },
      evasiClick: false,
      ticketsPreview: [],
      hdaid: null,
      modalDetail: false,
      isLoadingTicket: false,
      selectedRow: {}
    }
  },
  i18n: {
    messages: {
      it: {
        Modello: 'Modello',
        Brand: 'Brand',
        Tipo: 'Tipo',
        NumeroSeriale: 'Numero seriale',
        DataEvasione: 'Data evasione',
        PO: 'Ordine di acquisto',
        Bolla: 'Numero bolla',
        Garanzia: 'Garanzia',
        IndirizzoSpedizione: 'Indirizzo spedizione merce',
        DataConsegna: 'Data di consegna'
      },
      en: {
        Modello: 'Model',
        Brand: 'Brand',
        Tipo: 'Type',
        NumeroSeriale: 'Serial number',
        DataEvasione: 'Evasion date',
        PO: 'Purchase order',
        Bolla: 'Delivery doc',
        Garanzia: 'Warranty',
        IndirizzoSpedizione: 'Shipping address',
        DataConsegna: 'Delivery date'
      }
    }
  },
  mounted () {
    this.ElmecTableReadFromUrl()
    this.getBrand()
    this.getType()
    this.loadTicketsStatus()

  },
  watch: {
    downloadEvasi: function (value) {
      if (value === true) {
        this.loadTotal()
      }
    }
  },
  methods: {
    loadTotal: function () {
      this.$emit('disabled', true)
      this.$rhttp.get(process.env.VUE_APP_PURCHASE + '/mye/assetevasi?codicli=' + this.getUser().codiCli + '&page=1&per_page=20000').then(response => {
        if (response.status === 200) {
          this.listaTotal = response.body.data
          let colonneExport = [{val : 'descrizioneFamiglia', label: this.$t('Tipo')},
            {val :  'descrizioneParte', label: this.$t('Modello')},
            {val :  'descrizioneBrand', label: this.$t('Brand')},
            {val :  'numeroDiSerie', label: this.$t('NumeroSeriale')},
            {val :  'dataBolla', label: this.$t('DataEvasione')},
            {val :  'numeroPO', label: this.$t('PO')},
            {val :  'numeroBolla', label: this.$t('Bolla')}
          ]
          this.downloadCSV(this.listaTotal, colonneExport)
          this.$emit('no-disabled', false)
          this.$emit('evasi', false)
          this.$emit('click', false)
        }

      })
    },
    ModalSetPage: function (page) {
      this.modalParams = {
        page: page,
        per_page: 3
      }
      this.getTicket()
    },
    loadTicketsStatus: function () {
      this.$rhttp.get(process.env.VUE_APP_WS_SUPPORT + '/ticket/hda/codicli/' + this.getUser().codiCli + '/states/').then(response => {
        this.listaStatiComplete = []
        response.body.forEach((status) => {
          this.listaStatiComplete.push({
            val: status.user_state,
            label: status.user_state_description,
            idHda: status.stateID.trim()
          })
        })
      })
    },
    getDescStatus: function (statoID) {
      let state = ''
      this.listaStatiComplete.forEach(s => {
        if (s.idHda === statoID) {
          state = s.label
        }
      })
      return state
    },
    rowClicked: function (row) {
      this.selectedRow = row
      this.modalDetail = true
      this.getHdaId(row)
    },
    goToTicket: function (row) {
      this.$router.push({
        name: 'sup_incident_Dett',
        params: {
          IDTicket: row.ID,
          EIDTicket: row.EIDTicket
        }
      })
    },
    closeModalDetailAsset: function () {
      this.modalDetail = false
      this.ticketsPreview = []
      this.hdaid = null
    },
    fmtDate: function (date, fmt) {
      return moment(date).format(fmt ? fmt : 'DD-MM-YYYY ')
    },
    fmtCreation: function (date, fmt) {
      return moment(date).format(fmt ? fmt : 'DD-MM')
    },
    fmtLastUpdate: function (date, fmt) {
      return moment(date).format(fmt ? fmt : 'DD-MM-YYYY HH:mm:ss ')
    },
    async getHdaId (row) {
      this.ticketsPreview = []
      if (row.numeroDiSerie) {
        let params = {
          serialNumber: row.numeroDiSerie
        }
        try {
          let response = await this.$chttp.generic.get(process.env.VUE_APP_WS_SUPPORT + '/ticket/hda/codicli/' + this.getUser().codiCli + '/assets/', {params: params})
          if (response && response.status === 200) {
            if (response.body && response.body.length) {
              response.body.forEach((elem) => {
                this.hdaid = elem.ID
              })
            }
            this.getTicket()
          }
        } catch (err) {
          //err
        }
      } else {
        this.hdaid = null
      }
    },
    async getTicket () {
      this.modalParams['AssetID'] = this.hdaid
      this.isLoadingTicket = true
      let response = await this.$rhttp.get(process.env.VUE_APP_WS_SUPPORT + '/ticket/hda/codicli/' + this.getUser().codiCli + '/ticket/list/', {params: this.modalParams})
      if (response && response.status === 200) {
        this.responseModal = response.body
        this.ticketsPreview = response.body.data
      }
      this.isLoadingTicket = false
    },
    getBrand: function () {
      this.$rhttp.get(process.env.VUE_APP_PURCHASE + '/mye/assetevasi/brand?codicli=' + this.getUser().codiCli).then(response => {
        let data = response.body
        this.brandOptions = data
      })
    },
    getType: function () {
      this.$rhttp.get(process.env.VUE_APP_PURCHASE + '/mye/assetevasi/famiglia?codicli=' + this.getUser().codiCli).then(response => {
        let data = response.body
        this.typeOptions = data
      })
    },
    async downloadPDF (props) {
      if (this.isDemo) {
        return
      }
      this.isLoading = true
      let url = process.env.VUE_APP_RADAR_MYE + '/bolla/allegato?id_documento=' + props.idUnivocoBolla + '&codicli=' + this.getUser().codiCli
      let _this = this
      await this.$http.get(url).then((response) => {
        if (response.status === 200) {
          this.getAccountingDocument(response.body.base64Encoding, true, props.numeroBolla)
          this.traceSessionEvent("Purchase", "Asset evasi - click export", 'export asset evasi')
        }
        this.isLoading = false
      }, err => {
        if (err.status === 404) {
          _this.$toasted.show(_this.$t('non è presente un pdf per questa bolla'),
              {
                theme: 'primary',
                position: 'top-right',
                duration: 3000,
                type: 'success'
              })
        }
        this.isLoading = false
      })
    },
    setFormatDate: function (date) {
      return moment(date).format('DD-MM-YYYY')
    },
    dateFilterChangedHandler: function (val) {
      this.ElmecTableFilterChangeHandler(val)
    },
    filterChange (val) {
      let formatter = {
      }
      for (let key in val) {
        if (key === 'descrizioneParte') {
          formatter = {
            'descrizione_parte': val[key]
          }
        }
        if (key === 'descrizioneBrand') {
          formatter = {
            'brand': val[key]
          }
        }
        if (key === 'descrizioneFamiglia') {
          formatter = {
            'famiglia': val[key]
          }
        }
        if (key === 'numeroDiSerie') {
          formatter = {
            'serial_number': val[key]
          }
        }
        if (key === 'numeroPO') {
          formatter = {
            'rif_po': val[key]
          }
        }
      }
      this.ElmecTableMoreParams = {...this.ElmecTableMoreParams, ...formatter, ...{ page: 1 }}
    }
  }
}
</script>

<style lang="less">
@import "../../../style/less/variabili.less";
@import "../../../style/less/font.less";

#asset-evasi {
  padding-top: 20px;
  height: 100%;

  .click {
    cursor: pointer;
    .set-font-semi-bold(@size: 12px; @color: @myBlue);
    text-decoration: underline;
  }

  .no-ticket {
    text-align: center;
    margin-top: 20px;
  }

  .link-download {
    cursor: pointer;
    border: 1px solid @myBorderDisabled;
    background-color: @myLightGray;
    border-radius: 4px;
    margin-left: 20px;
    width: 33px;
    height: 22px;
    text-align: center;
    padding-top: 1px;
  }

  .detail {
    margin-bottom: 20px;
  }

  .description {
    font: var(--font-default);
    font-size: 12px;
  }

  .separator {
    border-bottom: lightgrey 1px solid
  }

  .title-ticket {
    margin-top: 20px;
    font-size: 15px
  }
}


</style>
