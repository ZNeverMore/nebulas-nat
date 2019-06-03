<template>
  <div class="hello">
    <h2>Essential Links</h2>
    <button type="button" @click="getTX">get transaction</button>
    <button @click="getNAT">get NAT</button>
  </div>
</template>

<script>

  import axios from 'axios'

  export default {

    name: 'HelloWorld',
    data() {
      return {
        pledgeList: [],
        nrList: [],
        allTX: [],
        nat: 0
      }
    },
    methods: {
      getTX() {
        this.pledgeList = []
        this.nrList = []
        this.allTX = []
        let address1 = 'n1ih5HkecJj93yx3vHtybRxoF8UEfdksxMi'
        let address = 'n1obU14f6Cp4Wv7zANVbtmXKNkpKCqQDgDM'
        let url = 'https://explorer-backend.nebulas.io/api/address/' + address1
        axios.get(url).then(response => {
          console.log(response)
          for (let i = 0; i < response.data.data.txList.length; i++) {
            let item = response.data.data.txList[i]
            if (item.status === 1) {
              let type = JSON.parse(item.data)
              if (type.function === 'triggerPledge') {
                this.pledgeList.push({
                  hash: item.hash,
                  timestamp: item.timestamp,
                  type: 'pledge'
                })
              }
              if (type.Function === 'triggerPledge') {
                this.pledgeList.push({
                  hash: item.hash,
                  timestamp: item.timestamp,
                  type: 'pledge'
                })
              }
              if (type.function === 'triggerNR') {
                this.nrList.push({
                  hash: item.hash,
                  timestamp: item.timestamp,
                  type: 'nr'
                })
              }
              if (type.Function === 'triggerNR') {
                this.nrList.push({
                  hash: item.hash,
                  timestamp: item.timestamp,
                  type: 'nr'
                })
              }
            }
          }
          this.allTX = this.pledgeList.concat(this.nrList)
          if (this.pledgeList.length > 0) {
            alert('success')
          } else {
            alert('failed')
          }
        })
      },
      getNAT() {
        let start = new Date('2019-5-20 12:00:00')
        let startTimestamp = start.getTime()
        let end = new Date('2019-5-27 12:00:00')
        let endTimestamp = end.getTime()
        let url = 'https://mainnet.nebulas.io/v1/user/getEventsByHash'
        for (let i = 0; i < this.pledgeList.length; i++) {
          let timestamp = this.pledgeList[i].timestamp
          // if (timestamp > startTimestamp && timestamp < endTimestamp) {
            let obj = {
              hash: this.pledgeList[i].hash
            }
            axios.post(url,
              JSON.stringify(obj),
              {
                headers: {
                  'Content-Type': 'application/json'
                }
              }).then(response => {
              let list = response.data.result.events
              for (let j = 0; j < list.length; j++) {
                if (list[j].topic === 'chain.contract.pledge') {
                  let json = JSON.parse(list[j].data)
                  for (let k = 0; k < json.data.length; k++) {
                    this.nat += parseFloat(json.data[k].nat)
                  }
                  break
                }
                if (list[j].topic === 'chain.contract.nr') {
                  let json = JSON.parse(list[j].data)
                  for (let k = 0; k < json.data.length; k++) {
                    this.nat += parseFloat(json.data[k].nat)
                  }
                  break
                }
              }
              console.log('nat:  ', this.nat)
            })
          // }
        }
        // let obj = {
        //   hash: '21c9f3007c30f59a6935fbe2a7f36fbad6a35dd237393ce4aa4d0fa8fd4da00c'
        // }
        // axios.post(url,
        //   JSON.stringify(obj),
        //   {
        //     headers: {
        //       'Content-Type': 'application/json'
        //     }
        //   }).then(response => {
        //   let list = response.data.result.events
        //   for (let j = 0; j < list.length; j++) {
        //     if (list[j].topic === 'chain.contract.pledge') {
        //       let json = JSON.parse(list[j].data)
        //       let totalNat = 0
        //       for (let k = 0; k < json.data.length; k++) {
        //         totalNat += parseFloat(json.data[k].nat)
        //       }
        //       console.log(totalNat)
        //       break
        //     }
        //   }
        // })
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }
</style>
