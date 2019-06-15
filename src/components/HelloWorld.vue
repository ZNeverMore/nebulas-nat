<template>
  <div class="hello">
    <div class="total-nat">
      <p v-show="allNat.show">
        {{allNat.str}}
      </p>
    </div>
    <div class="pledge-data">
      <el-date-picker
        v-model="pledgeData.period"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
      >
      </el-date-picker>
      <el-button type="primary" @click="getPledgeNatOrNrNat('pledge', pledgeData.period)" round>get pledge nat
      </el-button>
      <div>
        <p v-show="pledgeData.show">
          {{pledgeData.str}}
        </p>
      </div>
    </div>
    <div class="nr-data" style="margin-top: 50px">
      <el-date-picker
        v-model="nrData.period"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
      >
      </el-date-picker>
      <el-button type="primary" @click="getPledgeNatOrNrNat('nr', nrData.period)" round>get nr NAT</el-button>
      <div>
        <p v-show="nrData.show">
          {{nrData.str}}
        </p>
      </div>
    </div>
    <div class="vote-data" style="margin-top: 50px">
      <el-date-picker
        v-model="voteData.period"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
      >
      </el-date-picker>
      <el-button type="primary" @click="getVoteNat(voteData.period)" round>get vote nat</el-button>
      <div>
        <p v-show="voteData.show">
          {{voteData.str}}
        </p>
      </div>
    </div>
    <div class="address-data" style="margin-top: 50px">
      <el-date-picker
        v-model="addressData.period"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
        style="width: 350px"
      >
      </el-date-picker>
      <el-input v-model="addressData.address" placeholder="enter address" style="width: 150px"></el-input>
      <el-button type="primary" @click="getAddressData" round>get address data</el-button>
      <div>
        <p v-show="addressData.show">
          {{addressData.nr.str}}
          {{addressData.pledge.str}}
          {{addressData.vote.str}}
        </p>
      </div>
    </div>
    <div class="all-nat" style="margin-top: 50px">
      <el-date-picker
        v-model="allNat.periodData.period"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
        style="width: 350px"
      >
      </el-date-picker>
      <el-button type="primary" @click="getAllNat(allNat.periodData.period)" round>get all nat</el-button>
      <div>
        <p v-show="allNat.periodData.show">
          {{allNat.periodData.str}}
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
        testPeriod: [],
        pledgeData: {
          period: [],
          pledgeTxList: [],
          pledgeNat: 0,
          show: false,
          topic: 'chain.contract.NATToken',
          str: ''
        },
        nrData: {
          period: [],
          nrTxList: [],
          nrNat: 0,
          show: false,
          topic: 'chain.contract.NATToken',
          str: ''
        },
        voteData: {
          period: [],
          voteTxList: [],
          rewardNat: 0,
          burnNat: 0,
          show: false,
          topic: 'chain.contract.vote',
          str: ''
        },
        addressData: {
          address: '',
          period: [],
          pledge: {
            nat: 0,
            str: ''
          },
          nr: {
            nat: 0,
            str: ''
          },
          vote: {
            rewardNat: 0,
            usedNat: 0,
            burnNat: 0,
            str: ''
          },
          show: false
        },
        allNat: {
          periodData: {
            period: [],
            pledgeNat: 0,
            nrNat: 0,
            voteRewardNat: 0,
            totalNat: 0,
            str: '',
            show: false
          },
          pledgeNat: 0,
          nrNat: 0,
          voteRewardNat: 0,
          totalNat: 0,
          show: false,
          str: ''
        }
      }
    },
    mounted() {
      this.get(this.getTxUrl, this.pledgeAndNrAddress)
      this.get(this.getTxUrl, this.voteAddress)
      this.sleep(5000).then(() => {
      this.getAllNat([])
      })
    },
    methods: {
      get(url, address) {
        let firstParam = {
          params: {
            a: address,
            p: 1
          }
        }
        axios.get(url, firstParam).then(response => {
          let totalPage = response.data.data.totalPage
          for (let i = 1; i <= totalPage; i++) {
            let params = {
              params: {
                a: address,
                p: i
              }
            }
            axios.get(url, params).then(response => {
              let list = response.data.data.txnList
              for (let j = 0; j < list.length; j++) {
                if (list[j].status === 1) {
                  let json = JSON.parse(list[j].data)
                  if (json.Function === 'triggerNR') {
                    let item = {
                      hash: list[j].hash,
                      timestamp: list[j].timestamp
                    }
                    this.nrData.nrTxList.push(item)
                  }
                  if (json.function === 'triggerPledge' || json.Function === 'triggerPledge') {
                    let item = {
                      hash: list[j].hash,
                      timestamp: list[j].timestamp
                    }
                    this.pledgeData.pledgeTxList.push(item)
                  }
                  if (json.Function === 'vote') {
                    let argsList = JSON.parse(json.Args)
                    let voteUsedNat = parseInt(argsList[argsList.length - 1] / 1000000000000000000)
                    let item = {
                      hash: list[j].hash,
                      timestamp: list[j].timestamp,
                      usedNat: voteUsedNat
                    }
                    this.voteData.voteTxList.push(item)
                  }
                }
                if (i === totalPage && j === list.length - 1) {
                  console.log('get complete')
                }
              }
            })
          }
        })
      },
      getPledgeNatOrNrNat(tag, period, address = '') {
        if (period.length !== 2) {
          this.$notify.error({
            title: 'failed',
            message: 'select time first',
            type: 'error',
            duration: 1000
          })
          return
        }
        let list
        let topic
        let startTimestamp = period[0].getTime()
        let endTimestamp = period[1].getTime()
        let periodList = []
        if (tag === 'pledge') {
          list = this.pledgeData.pledgeTxList
          topic = this.pledgeData.topic
          this.pledgeData.pledgeNat = 0
          for (let m = 0; m < list.length; m++) {
            if (list[m].timestamp > startTimestamp && list[m].timestamp < endTimestamp) {
              periodList.push(list[m])
            }
          }
        }
        if (tag === 'nr') {
          list = this.nrData.nrTxList
          topic = this.nrData.topic
          this.nrData.nrNat = 0
          for (let n = 0; n < list.length; n++) {
            if (list[n].timestamp > startTimestamp && list[n].timestamp < endTimestamp) {
              periodList.push(list[n])
            }
          }
        }
        for (let i = 0; i < periodList.length; i++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: periodList[i].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            let events = response.data.result.events
            for (let j = 0; j < events.length; j++) {
              if (events[j].topic === topic) {
                let result = JSON.parse(events[j].data)
                let data = result.Produce.data
                for (let k = 0; k < data.length; k++) {
                  if (tag === 'pledge') {
                    if (address.length === 0) {
                      this.pledgeData.pledgeNat += parseFloat(data[k].value)
                    }
                    if (address.length > 0 && address === data[k].addr) {
                      this.addressData.pledge.nat += parseFloat(data[k].value)
                    }
                  }
                  if (tag === 'nr') {
                    if (address.length === 0) {
                      this.nrData.nrNat += parseFloat(data[k].value)
                    }
                    if (address.length > 0 && address === data[k].addr) {
                      this.addressData.nr.nat += parseFloat(data[k].value)
                    }
                  }
                }
              }
            }
          })
        }
        this.sleep(3000).then(() => {
          let startTime = this.dateFormat(startTimestamp)
          let endTime = this.dateFormat(endTimestamp)
          let startTimeMinus7 = this.dateFormat(startTimestamp - 3600 * 1000 * 24 * 7)
          let endTimeMinus7 = this.dateFormat(endTimestamp - 3600 * 1000 * 24 * 7)
          if (tag === 'nr' && address.length === 0) {
            this.nrData.str = ''
            this.nrData.show = false
            this.nrData.str = 'NR计算周期: ' + startTime + '-' + endTime + '\n\\'
              + ', 通过NR发放的NAT数量: ' + parseInt(this.nrData.nrNat / 1000000000000000000)
            this.nrData.show = true
          }
          if (tag === 'pledge' && address.length === 0) {
            this.pledgeData.str = ''
            this.pledgeData.show = false
            this.pledgeData.str = '质押周期: ' + startTimeMinus7 + '-' + endTimeMinus7
              + ', 通过该周期质押进行发放的NAT数量共有: ' + parseInt(this.pledgeData.pledgeNat / 1000000000000000000)
            this.pledgeData.show = true
          }
          if (address.length > 0) {
            this.addressData.pledge.str = ''
            this.addressData.nr.str = ''
            this.addressData.show = false
            this.addressData.nr.str = '地址: ' + address + ', 在NR计算周期: ' + startTime + '-' + endTime + '\n\\'
              + ', 通过NR发放的NAT数量: ' + parseInt(this.addressData.nr.nat / 1000000000000000000) + '\n\\'
            this.addressData.pledge.str = '质押周期: ' + startTimeMinus7 + '-' + endTimeMinus7
              + ', 通过该周期质押进行发放的NAT数量有: ' + parseInt(this.addressData.pledge.nat / 1000000000000000000)
          }
        })
      },
      getVoteNat(period, address = '') {
        if (period.length !== 2) {
          this.$notify.error({
            title: 'failed',
            message: 'select time first',
            type: 'error',
            duration: 1000
          })
          return
        }
        let list = this.voteData.voteTxList
        let topic = this.voteData.topic
        this.voteData.rewardNat = 0
        this.voteData.burnNat = 0
        let startTimestamp = period[0].getTime()
        let endTimestamp = period[1].getTime()
        let periodList = []
        for (let i = 0; i < list.length; i++) {
          if (list[i].timestamp > startTimestamp && list[i].timestamp < endTimestamp) {
            periodList.push(list[i])
          }
        }
        console.log('periodList: ', periodList)
        for (let j = 0; j < periodList.length; j++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: periodList[j].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            console.log('vote events: ', response)
            let events = response.data.result.events
            for (let k = 0; k < events.length; k++) {
              if (events[k].topic === topic) {
                let result = JSON.parse(events[k].data)
                let voteBurnNatList = result.data
                if (address.length === 0) {
                  this.voteData.rewardNat += parseFloat(result.reward)
                  this.voteData.burnNat += parseFloat(voteBurnNatList[1].nat)
                }
                if (address.length > 0 && address === voteBurnNatList[1].addr) {
                  this.addressData.vote.burnNat += parseFloat(voteBurnNatList[1].nat)
                  this.addressData.vote.rewardNat += parseFloat(result.reward)
                  this.addressData.vote.usedNat += periodList[j].usedNat
                }
              }
            }
          })
        }
        this.sleep(3000).then(() => {
          let startTime = this.dateFormat(startTimestamp)
          let endTime = this.dateFormat(endTimestamp)
          if (address.length === 0) {
            this.voteData.str = ''
            this.voteData.show = false
            this.voteData.str = '投票周期: ' + startTime + '-' + endTime
              + ', 本周已投出的NAT中获得激励的数量: ' + parseInt(this.voteData.rewardNat / 10)
            this.voteData.show = true
          }
          if (address.length > 0) {
            this.addressData.vote.str = ''
            this.addressData.vote.str = '投票周期: ' + startTime + '-' + endTime
              + ', 本周已投出的NAT中获得激励的数量: ' + parseInt(this.addressData.vote.rewardNat / 10)
            this.addressData.show = true
          }
        })

      },
      getAddressData() {
        if (this.addressData.period.length !== 2) {
          this.$message({
            type: 'error',
            message: 'please select the time',
            duration: 1000
          })
          return
        }
        if (this.addressData.address.length === 0 || this.addressData.address === undefined) {
          this.$message({
            type: 'error',
            message: 'please enter the address',
            duration: 1000
          })
          return
        }
        let period = this.addressData.period
        let address = this.addressData.address
        this.getPledgeNatOrNrNat('pledge', period, address)
        this.getPledgeNatOrNrNat('nr', period, address)
        this.getVoteNat(period, address)
      },
      getAllNat(period = []) {
        let pledgeTxList = this.pledgeData.pledgeTxList
        let nrTxList = this.nrData.nrTxList
        let voteTxList = this.voteData.voteTxList
        if (pledgeTxList.length < 0 || nrTxList.length < 0 || voteTxList, length < 0) {
          this.$notify.error({
            title: 'failed',
            message: 'get failed',
            type: 'error',
            duration: 1000
          })
          return
        }
        this.allNat.pledgeNat = 0
        this.allNat.nrNat = 0
        this.allNat.voteRewardNat = 0
        this.allNat.totalNat = 0
        if (period.length === 2) {
          this.allNat.periodData.totalNat = 0
          this.allNat.periodData.pledgeNat = 0
          this.allNat.periodData.nrNat = 0
          this.allNat.voteRewardNat = 0
          pledgeTxList = this.filterList(pledgeTxList, period)
          nrTxList = this.filterList(nrTxList, period)
          voteTxList = this.filterList(voteTxList, period)
        }
        for (let i = 0; i < pledgeTxList.length; i++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: pledgeTxList[i].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            console.log('pledgeTxList.response: ', response)
            let events = response.data.result.events
            for (let j = 0; j < events.length; j++) {
              if (events[j].topic === this.pledgeData.topic) {
                let result = JSON.parse(events[j].data)
                let data = result.Produce.data
                for (let k = 0; k < data.length; k++) {
                  if (period.length === 0) {
                    this.allNat.pledgeNat += parseFloat(data[k].value)
                    console.log('plus allNat.pledgeNat')
                  }
                  if (period.length === 2) {
                    this.allNat.periodData.pledgeNat += parseFloat(data[k].value)
                  }
                }
              }
            }
          });
        }
        for (let a = 0; a < nrTxList.length; a++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: nrTxList[a].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            let events = response.data.result.events
            for (let b = 0; b < events.length; b++) {
              if (events[b].topic === this.nrData.topic) {
                let result = JSON.parse(events[b].data)
                let data = result.Produce.data
                for (let c = 0; c < data.length; c++) {
                  if (period.length === 0) {
                    this.allNat.nrNat += parseFloat(data[c].value)
                  }
                  if (period.length === 2) {
                    this.allNat.periodData.nrNat += parseFloat(data[c].value)
                  }
                }
              }
            }
          })
        }
        for (let z = 0; z < voteTxList.length; z++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: voteTxList[z].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            let events = response.data.result.events
            for (let x = 0; x < events.length; x++) {
              if (events[x].topic === this.voteData.topic) {
                let result = JSON.parse(events[x].data)
                if (period.length === 0) {
                  this.allNat.voteRewardNat += parseFloat(result.reward)
                }
                if (period.length === 2) {
                  this.allNat.periodData.voteRewardNat += parseFloat(result.reward)
                }

              }
            }
          })
        }
        this.sleep(5000).then(() => {
          if (period.length === 0) {
            this.allNat.str = ''
            this.allNat.show = false
            let pledgeNat = parseInt(this.allNat.pledgeNat / 1000000000000000000);
            let nrNat = parseInt(this.allNat.nrNat / 1000000000000000000)
            let voteRewardNat = parseInt(this.allNat.voteRewardNat / 10)
            this.allNat.totalNat = pledgeNat + nrNat + voteRewardNat
            this.allNat.str = '全网共发行NAT: ' + this.allNat.totalNat
            this.allNat.show = true
          }
          if (period.length === 2) {
            this.allNat.periodData.str = ''
            this.allNat.periodData.show = false
            let startTimestamp = period[0].getTime()
            let endTimestamp = period[1].getTime()
            let pledgeNat = parseInt(this.allNat.periodData.pledgeNat / 1000000000000000000);
            let nrNat = parseInt(this.allNat.periodData.nrNat / 1000000000000000000)
            let voteRewardNat = parseInt(this.allNat.periodData.voteRewardNat / 10)
            this.allNat.periodData.totalNat = pledgeNat + nrNat + voteRewardNat
            this.allNat.periodData.str = '计算周期: ' + this.dateFormat(startTimestamp) + '-' + this.dateFormat(endTimestamp)
              + ', 全网共发行的NAT: ' + this.allNat.periodData.totalNat
            this.allNat.periodData.show = true
          }
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
      },
      sleep(milliseconds) {
        return new Promise(resolve => setTimeout(resolve, milliseconds))
      },
      filterList(targetList, period) {
        let startTimestamp = period[0].getTime()
        let endTimestamp = period[1].getTime()
        let resultList = []
        for (let i = 0; i < targetList.length; i++) {
          if (targetList[i].timestamp > startTimestamp && targetList[i].timestamp < endTimestamp) {
            resultList.push(targetList[i])
          }
        }
        return resultList
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .vote-data {

  }
</style>
