<template>
  <div class="hello">
    <el-date-picker
      v-model="startAndEndTime"
      type="datetimerange"
      range-separator="至"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
    >
    </el-date-picker>
    <el-button type="primary" @click="getTX" round>get transaction</el-button>
    <el-button type="primary" @click="getNAT" round>get NAT</el-button>
    <div v-show="pledgeData.show">
      <p>
        {{nrData.str}}
      </p>
      <p>
        {{pledgeData.str}}
      </p>
    </div>
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
        totalPage: 0,
        startAndEndTime: [],
        pledgeData: {
          show: false
        },
        nrData: {
          show: false
        }
      }
    },
    methods: {
      getTX() {
        if (this.startAndEndTime.length !== 2) {
          alert('please select time')
          return
        }
        this.nrList = [];
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
          return
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
            let startTimestamp = this.startAndEndTime[0].getTime()
            let endTimestamp = this.startAndEndTime[1].getTime()
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
            // console.log(response)
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
            // console.log('pledge-response: ', response)
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

        function sleep(milliseconds) {
          return new Promise(resolve => setTimeout(resolve, milliseconds))
        }

        sleep(2000).then(() => {
          let nrStr = 'NR计算周期：' + this.dateFormat(this.startAndEndTime[0].getTime()) + '-' + this.dateFormat(this.startAndEndTime[1].getTime()) + '\n'
          + '通过NR发放的NAT数量：' + parseInt(this.nrNat / 1000000000000000000)
          this.nrData.str = nrStr
          this.nrData.show = true
          let pledgeStr = '质押周期：' + this.dateFormat(this.startAndEndTime[0].getTime() - 3600 * 1000 * 24 * 7) + '-' + this.dateFormat(this.startAndEndTime[1].getTime() - 3600 * 1000 * 24 * 7)
          + '通过该周期质押进行发放的NAT数量共有' + parseInt(this.pledgeNat / 1000000000000000000)
          this.pledgeData.str = pledgeStr
          this.pledgeData.show = true
        });
      },
      dateFormat(timestamp) {
        console.log('timestamp: ', timestamp)
        let date = new Date(timestamp)
        let month = date.getMonth() + 1
        let day = date.getDate()
        let hour = date.getHours()
        let minutes = date.getMinutes()
        return month + '月' + day + '日' + hour + ':' + minutes
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
