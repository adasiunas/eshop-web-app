<template>
  <el-container>
    <el-main>
      <status-bar class="gd_status_bar" v-if="this.$store.getters.isImportInProgress"></status-bar>
      <AdminNavigation></AdminNavigation>
      <router-view/>
    </el-main>
  </el-container>
</template>
<script>
  import AdminNavigation from '@/components/Navigation/AdminNavigation'
  import StatusBar from '@/components/Import/StatusBar'
  import EventBus from '@/eventBus/index.js'

  export default {
    components: {
      AdminNavigation,
      StatusBar
    },
    data () {
      return {
        file: []
      }
    },
    mounted () {
      EventBus.$on('importStarted', file => {
        this.importItems(file)
      })
      EventBus.$on('exportStarted', () => {
        window.onbeforeunload = () => {
          return 'export running'
        }
      })
      EventBus.$on('exportFinished', () => {
        window.onbeforeunload = undefined
      })
    },
    beforeDestroy () {
      EventBus.$off('importStarted', file => {
        this.importItems(file)
      })
      EventBus.$off('exportStarted', () => {
        window.onbeforeunload = () => {
          return 'export running'
        }
      })
      EventBus.$off('exportFinished', () => {
        window.onbeforeunload = undefined
      })
    },
    methods: {
      importItems (file) {
        this.$store.dispatch('startImport')
        window.onbeforeunload = () => {
          return 'export running'
        }
        var form = new FormData()
        form.append('file', file)
        this.axios.post('/admin/items/import', form)
          .then(Response => {
            if (Response.data.errors && Response.data.errors.length > 0) {
              this.$store.dispatch('setImportErrors', Response.data.errors)
              this.$notify({
                message: 'Import completed with errors',
                type: 'warning',
                offset: 50
              })
            } else {
              this.$notify({
                message: 'Import completed successfully',
                type: 'success',
                offset: 50
              })
            }
            this.$store.dispatch('setImportedItems', Response.data.items)
            window.onbeforeunload = undefined
            this.$store.dispatch('endImport')
          })
          .catch(err => {
            this.$store.dispatch('endImport')
            window.onbeforeunload = undefined
            if (err.cookieExpired) {
              EventBus.$emit('cookieExpired')
              return
            }
            if (err.response) {
              this.$notify({
                message: err.response.data.message,
                type: 'error',
                offset: 50
              })
            }
            this.$store.dispatch('setImportErrors', null)
            this.$store.dispatch('setImportedItems', null)
          })
      }
    }
  }

</script>

<style scoped>
  .el-header {
    padding: 0;
    width: 100%;
  }

  .el-main {
    padding: 16px 8px 0 16px;
  }

  header {
    height: 26px !important;
  }

  .gd_status_bar {
    position: absolute;
    top: 0px;
    right: 0px;
    left: 0px;
  }

</style>
