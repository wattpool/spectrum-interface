<template>
  <div>
    <b-row class="justify-content-md-center">
      <b-col md="10">
        <b-breadcrumb>
          <b-breadcrumb-item :to="{name: 'Home'}">Home</b-breadcrumb-item>
          <b-breadcrumb-item :active="type === 'latest'" :to="{name: 'Transactions', params: {type: 'latest'}}">Transactions</b-breadcrumb-item>
          <b-breadcrumb-item v-if="type === 'pending'" active>Pending</b-breadcrumb-item>
          <b-breadcrumb-item v-else-if="type === 'block'" active>Block {{ blockNumber }}</b-breadcrumb-item>
          <b-breadcrumb-link>
            <b-button :class="{fa: true, 'fa-refresh': true, 'fa-spin': refreshing, 'btn-breadcrumb': true}" v-on:click="fetch()"/>
          </b-breadcrumb-link>
        </b-breadcrumb>
        <TxnsTable :items="txns" :pending="type === 'pending'" :block="type === 'block'" :blockNumber="blockNumber" :total="total"/>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import axios from 'axios'
import TxnsTable from '../components/tables/Txns.vue'

export default {
  name: 'Transactions',
  props: ['type', 'blockNumber'],
  watch: {
    '$route': {
      handler: function (from, to) {
        this.fetch()
      },
      immediate: true
    },
    type: function () {
      this.fetch()
    }
  },
  data () {
    return {
      refreshing: false,
      txns: [],
      total: 0,
      errors: []
    }
  },
  methods: {
    fetch: function () {
      this.refreshing = true
      let self = this
      if (this.type === 'latest') {
        axios.get(this.$store.state.api + 'latesttransactions/1000')
          .then(response => {
            this.txns = response.data.txns
            this.total = response.data.total
            setTimeout(function () {
              self.refreshing = false
            }, 2000)
          })
          .catch(e => {
            this.errors.push(e)
          })
      } else if (this.type === 'block') {
        axios.get(this.$store.state.api + 'blocktransactions/' + this.blockNumber)
          .then(response => {
            this.txns = response.data
            setTimeout(function () {
              self.refreshing = false
            }, 2000)
          })
          .catch(e => {
            this.errors.push(e)
          })
      } else {
        // pending transactions
        axios.post(this.$store.state.rpc, {
          jsonrpc: '2.0',
          method: 'eth_getBlockByNumber',
          params: [
            'pending',
            true
          ],
          id: 1
        })
          .then(response => {
            this.txns = response.data.result.transactions
            setTimeout(function () {
              self.refreshing = false
            }, 2000)
          })
          .catch(e => {
            this.errors.push(e)
          })
      }
    }
  },
  components: {
    TxnsTable
  }
}
</script>
