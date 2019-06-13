<template>
  <div class="hello">
    <div class="pledge-nr-data">
      <el-date-picker
        v-model="startAndEndTime"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
      >
      </el-date-picker>
      <el-button type="primary" @click="getTX(startAndEndTime, getTxUrl, pledgeAndNrAddress)" round>get pledge&nr transaction</el-button>
      <el-button type="primary" @click="getNAT(startAndEndTime)" round>get NAT</el-button>
      <div>
        <p v-show="pledgeData.show">
          {{nrData.str}}
        </p>
        <p v-show="nrData.show">
          {{pledgeData.str}}
        </p>
      </div>
    </div>
    <div class="vote-data" style="margin-top: 50px">
      <!--<el-input v-model="inputAddress" style="width: 280px"></el-input>-->
      <el-date-picker
        v-model="voteTime"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
      >
      </el-date-picker>
      <el-button type="primary" @click="getTX(voteTime, getTxUrl, voteAddress)" round>get vote transaction</el-button>
      <el-button type="primary" @click="getVoteNAT(voteTime)" round>get vote nat</el-button>
      <div>
        <p v-show="voteData.show">
          {{voteData.str}}
        </p>
      </div>
    </div>
  </div>
</template>

<script>

  import axios from 'axios'

  export default {

    name: 'HelloWorld',
    data() {
      return {
        pledgeAndNrAddress: 'n1EoNsJNXG1tN3z9rvjwPKoBXbJMqAjmESC',
        voteAddress: 'n1pADU7jnrvpPzcWusGkaizZoWgUywMRGMY',
        getTxUrl: 'https://explorer-backend.nebulas.io/api/tx/',
        getEventByHashUrl: 'https://mainnet.nebulas.io/v1/user/getEventsByHash',
        inputAddress: '',
        totalPage: 0,
        startAndEndTime: [],
        voteTime: [],
        allTx: [],
        pledgeData: {
          pledgeList: [],
          pledgeNat: 0,
          show: false
        },
        nrData: {
          nrNat: 0,
          nrList: [],
          show: false
        },
        voteData: {
          voteList: [],
          show: false,
          rewardNat: 0,
          burnNat: 0
        }
      }
    },
    methods: {
      getTX(time, url, address) {
        if (time.length !== 2) {
          this.$message({
            message: 'please select time first',
            type: 'error',
            duration: 1000
          })
          return
        }
        this.nrData.nrList = []
        this.pledgeData.pledgeList = []
        this.voteData.voteList = []
        let firstParams = {
          params: {
            a: address,
            p: 1
          }
        }
        axios.get(url, firstParams).then(response => {
          let data = response.data
          this.totalPage = data.data.totalPage
          this.getAllTx(time, url, address)
        })
      },
      getAllTx(time, url, address) {
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
            let startTimestamp = time[0].getTime()
            let endTimestamp = time[1].getTime()
            let list = data.data.txnList
            for (let j = 0; j < list.length; j++) {
              if (list[j].status === 1) {
                let item = {
                  hash: list[j].hash,
                  timestamp: list[j].timestamp
                }
                this.allTx.push(item)
              }
              if (list[j].timestamp > startTimestamp && list[j].timestamp < endTimestamp && list[j].status === 1) {
                let json = JSON.parse(list[j].data);
                if (json.Function === 'triggerNR') {
                  let item = {
                    hash: list[j].hash,
                    timestamp: list[j].timestamp
                  };
                  this.nrData.nrList.push(item)
                }
                if (json.function === 'triggerPledge' || json.Function === 'triggerPledge') {
                  let item = {
                    hash: list[j].hash,
                    timestamp: list[j].timestamp
                  }
                  this.pledgeData.pledgeList.push(item)
                }
                if (json.Function === 'vote') {
                  let item = {
                    hash: list[j].hash,
                    timestamp: list[j].timestamp
                  }
                  this.voteData.voteList.push(item)
                }
              }
              if (i === this.totalPage && j === list.length - 1) {
                this.$notify({
                  title: 'success',
                  message: 'get transaction success',
                  type: 'success',
                  duration: 700
                })
              }
            }
          }).catch(function () {
            this.$notify.error({
              title: 'failed',
              message: 'get transaction failed',
              type: 'error',
              duration: 1000
            })
          })
        }
      },
      getNAT(time) {
        if (this.nrData.nrList.length === 0 || this.pledgeData.pledgeList.length === 0 || time.length !== 2) {
          this.$message({
            type: 'error',
            message: 'please get transaction first',
            duration: 1000
          })
          return
        }
        this.nrData.nrNat = 0
        this.pledgeData.pledgeNat = 0
        for (let i = 0; i < this.nrData.nrList.length; i++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: this.nrData.nrList[i].hash}),
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
                for (let k = 0; k < natList.length; k++) {
                  this.nrData.nrNat += parseFloat(natList[k].value)
                }
              }
            }
          })
        }

        for (let l = 0; l < this.pledgeData.pledgeList.length; l++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: this.pledgeData.pledgeList[l].hash}),
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
                  this.pledgeData.pledgeNat += parseFloat(pledgeNatList[n].value)
                }
              }
            }
          })
        }

        function sleep(milliseconds) {
          return new Promise(resolve => setTimeout(resolve, milliseconds))
        }

        sleep(2000).then(() => {
          this.nrData.str = ''
          this.nrData.show = false
          this.pledgeData.str = ''
          this.pledgeData.show = false
          let nrStr = 'NR计算周期：' + this.dateFormat(time[0].getTime()) + '-' + this.dateFormat(time[1].getTime()) + '\n'
            + ', 通过NR发放的NAT数量: ' + parseInt(this.nrData.nrNat / 1000000000000000000)
          this.nrData.str = nrStr
          this.nrData.show = true
          let pledgeStr = '质押周期：' + this.dateFormat(time[0].getTime() - 3600 * 1000 * 24 * 7) + '-' + this.dateFormat(time[1].getTime() - 3600 * 1000 * 24 * 7)
            + ', 通过该周期质押进行发放的NAT数量共有: ' + parseInt(this.pledgeData.pledgeNat / 1000000000000000000)
          this.pledgeData.str = pledgeStr
          this.pledgeData.show = true
        })
      },
      getVoteNAT(time) {
        if (this.voteData.voteList.length === 0 || time.length !== 2) {
          this.$message({
            type: 'error',
            message: 'please get vote transaction first',
            duration: 1000
          })
          return
        }
        this.voteData.rewardNat = 0
        this.voteData.burnNat = 0
        for (let i = 0; i < this.voteData.voteList.length; i++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: this.voteData.voteList[i].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            let voteEvents = response.data.result.events
            for (let j = 0; j < voteEvents.length; j++) {
              if (voteEvents[j].topic === 'chain.contract.vote') {
                let result = JSON.parse(voteEvents[j].data);
                this.voteData.rewardNat += parseFloat(result.reward)
                let voteBurnNatList = result.data
                this.voteData.burnNat += parseFloat(voteBurnNatList[1].nat)
              }
            }
          })
        }

        function sleep(milliseconds) {
          return new Promise(resolve => setTimeout(resolve, milliseconds))
        }

        sleep(2000).then(() => {
          this.voteData.str = ''
          this.voteData.show = false
          this.voteData.str = '投票周期: ' + this.dateFormat(time[0].getTime()) + '-' + this.dateFormat(time[1].getTime())
          + ', 本周已投出的NAT中获得激励的数量: ' + parseInt(this.voteData.rewardNat / 10)
          this.voteData.show = true
        })

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
  .vote-data {

  }
</style>
