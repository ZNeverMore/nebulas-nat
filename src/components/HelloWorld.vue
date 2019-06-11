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
        nrNat: 0,
        pledgeNat: 0,
        nrList: [],
        pledgeList: [],
        totalPage: 0
      }
    },
    methods: {
      getTX() {
        this.nrList = []
        this.pledgeList = []
        let address = 'n1EoNsJNXG1tN3z9rvjwPKoBXbJMqAjmESC'
        let url = 'https://explorer-backend.nebulas.io/api/tx/'
        let firstParams = {
          params: {
            a: address,
            p: 1
          }
        }
        axios.get(url, firstParams).then(response => {
          let data = response.data
          this.totalPage = data.data.totalPage
          this.getAllTx(url, address)
        })
      },
      getAllTx(url, address) {
        if (this.totalPage === 0) {
          alert('illegal totalPage')
        }
        for (let i = 1; i <= this.totalPage; i++) {
          let params = {
            params: {
              a: address,
              p: i
            }
          }
          axios.get(url, params).then(response => {
            console.log(response.data)
            let data = response.data
            this.totalPage = data.data.totalPage
            let start = new Date('2019-5-27 12:00:00')
            let startTimestamp = start.getTime()
            let end = new Date('2019-6-4 12:00:00')
            let endTimestamp = end.getTime()
            let list = data.data.txnList
            for (let j = 0; j < list.length; j++) {
              if (list[j].timestamp > startTimestamp && list[j].timestamp < endTimestamp && list[j].status === 1) {
                console.log()
                let json = JSON.parse(list[j].data)
                if (json.Function === 'triggerNR') {
                  let item = {
                    hash: list[j].hash,
                    timestamp: list[j].timestamp
                  };
                  this.nrList.push(item)
                }
                if (json.function === 'triggerPledge' || json.Function === 'triggerPledge') {
                  let item = {
                    hash: list[j].hash,
                    timestamp: list[j].timestamp
                  }
                  this.pledgeList.push(item)
                }
              }
            }
          })
        }
      },
      getNAT() {
        if (this.nrList.length === 0) {
          alert('please get transaction first')
          return
        }
        let count = 0;
        let url = 'https://mainnet.nebulas.io/v1/user/getEventsByHash';
        for (let i = 0; i < this.nrList.length; i++) {
          axios.post(url,
            JSON.stringify({hash: this.nrList[i].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            console.log(response)
            let events = response.data.result.events
            for (let j = 0; j < events.length; j++) {
              if (events[j].topic === 'chain.contract.NATToken') {
                let result = JSON.parse(events[j].data)
                let natList = result.Produce.data
                count += natList.length;
                for (let k = 0; k < natList.length; k++) {
                  this.nrNat += parseFloat(natList[k].value)

                }
              }
            }
            console.log("total:" + count)
          })
        }

        for (let l = 0; l < this.pledgeList.length; l++) {
          axios.post(url,
          JSON.stringify({hash: this.pledgeList[l].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
              console.log('pledge-response: ', response)
            let pledgeEvents = response.data.result.events
            for (let m = 0; m < pledgeEvents.length; m++) {
              if (pledgeEvents[m].topic === 'chain.contract.NATToken') {
                let result = JSON.parse(pledgeEvents[m].data)
                let pledgeNatList = result.Produce.data
                for (let n = 0; n < pledgeNatList.length; n++) {
                  this.pledgeNat += parseFloat(pledgeNatList[n].value)
                }
              }
            }
          })
        }


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
