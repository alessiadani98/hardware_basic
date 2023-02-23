<template>
  <div id="asset-in-evasione">
    <div slot="body" class="chart-wrapper flex-height-section">
      <img v-if="loadingTable" style="height: 40px" src="/icons/loader.svg">
      <elmec-table
          v-if="!loadingTable"
          class="use-remaining-height-space"
          :data="dataInEvasione"
          :loading="loadingTable"
          height="100%"
          size="mini"
      >
        <elmec-table-column :label=" $t('Modello') " prop="descrizioneArticolo">
        </elmec-table-column>
        <elmec-table-column :label=" $t('Brand') " prop="descrizioneBrand" :width="'160'">
        </elmec-table-column>
        <elmec-table-column :label="$t('Quantità')" prop="qtaOrdinata" :width="'100'">
          <template slot-scope="scope">
            <div style="text-align: center">
              {{scope.row.qtaOrdinata}}
            </div>
          </template>
        </elmec-table-column>
        <elmec-table-column :label="$t('NumeroOrdine')" prop="numeroOrdine"  :width="'140'">
        </elmec-table-column>
        <elmec-table-column :label="$t('Stato') " prop="stato" :filter-method="filterHandler" :width="'150'">
          <template v-slot:header="scope">
            <elmec-table-filter
                v-model="tableFilter.state"
                :column-scope="scope"
                :placement="'bottom-start'"
                :options="filterOptions"
                type="choice"
                :filtered-by-api="false"
            ></elmec-table-filter>
          </template>
          <template slot-scope="scope">
            <div style="color: orange;cursor: pointer!important;"
                 v-if="scope.row.stato === 'IN PROGRESS'"
                 v-tooltip.top="{ content: (scope.row.stato === 'IN PROGRESS' && scope.row.consegnaOFdaOC  === null &&
                  scope.row.consegnaOCConBID === null && scope.row.consegnaAltriOF === null? $t('ToManage'): $t('ToManageDate') + dataConsegnaToManage) }">
              {{ mappingState[scope.row.stato] }}
            </div>
            <div style="color: orange;cursor: pointer!important;"
                 v-if="scope.row.stato === 'TO BE STAGED'"
                 v-tooltip.top="{ content: ($t('WorkInProgress')) }">
              {{ mappingState[scope.row.stato] }}
            </div>
            <div v-else-if="scope.row.stato === 'CANCELED'" class="cancelled">
              {{ mappingState[scope.row.stato] }}
            </div>
            <div v-else-if="scope.row.stato === ''" style="cursor: pointer!important;"
                 v-tooltip.top="{ content: (mappingState[scope.row.stato] === 'Unavailable'?  $t('Unavailable'):
                  $t('WaitingForSupplier') + setFormatDate(dataConsegna))} "
            >
              {{ mappingState[scope.row.stato] }}
            </div>
          </template>
        </elmec-table-column>
        <elmec-table-column :label=" $t('PO')" prop="numeroPO" :width="'170'">
          <template slot-scope="scope">
            <div v-if="scope.row.numeroPO">
              {{ scope.row.numeroPO }}
            </div>
            <div v-else> -</div>
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
  </div>
</template>

<script>
import {
  ElmecTable,
  ElmecTableColumn,
  ElmecTablePagination,
  ElmecTableFilter,
  ElmecTablePaginationMixin
} from '@elmecglobal/elmec-ui'
import moment from 'moment'

export default {
  name: 'AssetInEvasione',
  components: {
    ElmecTable,
    ElmecTableColumn,
    ElmecTablePagination,
    ElmecTableFilter
  },
  mixins: [ElmecTablePaginationMixin],
  props: {
    downloadEvasi: {}
  },
  data () {
    return {
      ElmecTableUrl: process.env.VUE_APP_PURCHASE + '/mye/assetinevasione?codicli=' + this.getUser().codiCli,
      ElmecTableDefaultParams: {
      },
      loadingTable: false,
      ElmecTableUrlFilter: true,
      tableFilter: {
        state: null
      },
      listaStatiComplete: [],
      filterOptions: [],
      brandOptions: [],
      listaTotal: [],
      dataConsegna: null,
      dataConsegnaToManage: null,
      mappingState: {
      },
      dataInEvasione: [],
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
        Quantità: 'Quantità',
        PO: 'Ordine di acquisto',
        DataConsegna: 'Data di consegna',
        NumeroOrdine:'Numero ordine',
        ToManage: 'Merce da prendere in carico, ma disponibile nel magazzino',
        ToManageDate: 'La merce è da prendere in carico e sarà gestita indicativamente il ',
        WorkInProgress: 'La merce è in fase di preparazione',
        Unavailable: 'Al momento la merce non è disponibile e non sappiamo quando diventerà disponibile ',
        WaitingForSupplier: 'La merce verrà consegnata nei magazzini Elmec presumibilmente alla data indicata ',
        Stato: 'Stato'
      },
      en: {
        Modello: 'Model',
        Brand: 'Brand',
        Tipo: 'Type',
        ToManage: 'Wares to take in charge, but available in the warehouse',
        ToManageDate: 'Wares to take in charge, and will be managed approximately on ',
        WorkInProgress: 'Wares are being prepared',
        Unavailable: 'At the moment the wares are not available and we don\'t know when they will become available',
        WaitingForSupplier: 'The goods will be delivered to the Elmec warehouses presumably on the indicated date ',
        Quantità: 'Quantity',
        PO: 'Purchase order',
        DataConsegna: 'Delivery date',
        NumeroOrdine:'Order number',
        Stato: 'Status'
      }
    }
  },
  mounted () {
    this.tableDataInEvasione()
  },
  watch: {
    downloadEvasi: function (value) {
      if (value === true) {
        this.loadTotal()
      }
    }
  },
  methods: {
    setFormatDate: function (date) {
      return moment(date).format('DD-MM-YYYY')
    },
    loadTotal: function () {
      this.$emit('disabled', true)
          let colonneExport = [
            {val: 'descrizioneArticolo', label: this.$t('Modello')},
            {val: 'descrizioneBrand', label: this.$t('Brand')},
            {val: 'qtaOrdinata', label: this.$t('Quantità')},
            {val: 'consegnaOFdaOC', label: this.$t('DataConsegna')},
            {val: 'numeroOrdine', label: this.$t('NumeroOrdine')},
            {val: 'stato', label: this.$t('Stato')},
            {val: 'numeroPO', label: this.$t('PO')}
          ]
          this.downloadCSV(this.dataInEvasione, colonneExport)
          this.$emit('no-disabled', false)
          this.$emit('evasi', false)
          this.$emit('click', false)

    },
    tableDataInEvasione: function () {
      this.loadingTable= true
      this.$rhttp.get(process.env.VUE_APP_PURCHASE + '/mye/assetinevasione?codicli=' + this.getUser().codiCli).then(response => {
        let statiPresenti = []
        if (response.status === 200) {
          let resp = response.body
          resp.forEach((val) =>{
            if (val.stato !== 'DELIVERED') {
              this.dataInEvasione.push(val)
              let filter = []
              this.dataInEvasione.forEach((el) => {
                if (el.consegnaAltriOF !== null || el.consegnaOCConBID !== null || el.consegnaOFdaOC!== null) {
                  this.dataConsegna = el.consegnaAltriOF ? el.consegnaAltriOF : (el.consegnaOCConBID && el.consegnaOFdaOC === null ? el.consegnaOCConBID : el.consegnaOFdaOC)
                  let dataManage = el.consegnaAltriOF ? el.consegnaAltriOF : (el.consegnaOCConBID && el.consegnaOFdaOC === null ? el.consegnaOCConBID :
                      (el.consegnaOFdaOC !== null && el.giacenza === 0) ? el.consegnaOFdaOC : null )
                  if (dataManage !== null) {
                    this.dataConsegnaToManage = this.setFormatDate(dataManage)
                  }
                }
                this.mappingState = {
                  'IN PROGRESS': 'To Manage',
                  '': el.stato === '' && (el.consegnaAltriOF === null || el.consegnaOCConBID === null || el.consegnaOFdaOC === null) ? 'Unavailable' : 'Waiting for supplier',
                  'CANCELED': 'Cancelled',
                  'TO BE STAGED': ' Work in progress'
                }
              })
              statiPresenti.push(val.stato)
              for (let key in this.mappingState) {
                if (statiPresenti.includes(key)) {
                  filter.push({label: this.mappingState[key], value: key})
                  this.filterOptions = filter
                }
              }
            }
          })
          this.loadingTable= false
        }
      })
    },
    filterHandler (value, row, column) {
      return value !== null ? row[column.property] === value : true
    },
    ModalSetPage: function (page) {
      this.modalParams = {
        page: page,
        per_page: 3
      }
      this.getTicket()
    },
    fmtDate: function (date, fmt) {
      return moment(date).format(fmt ? fmt : 'DD-MM-YYYY ')
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
          this.traceSessionEvent("Purchase", "Asset in evasione - click export", 'export asset in evasione')
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
    }
  }
}
</script>

<style lang="less">
@import "../../../style/less/variabili.less";
@import "../../../style/less/font.less";

#asset-in-evasione {
  padding-top: 20px;
  height: 100%;
  .delivered {
    color: #00AC50;
  }
  .cancelled {
    color: red;
  }
  .elmec-ui.el-table.el-table--enable-row-hover .el-table__body tr > td {
    cursor: default !important;
    background-color: unset !important;
  }
  body .el-table.el-table--enable-row-hover .el-table__body tr > td {
    cursor: default!important;
    background-color: unset !important;
  }

}


</style>
