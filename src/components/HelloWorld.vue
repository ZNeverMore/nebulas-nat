<template>
  <div class="hello">
    <div class="pledge-data">
      <el-date-picker
        v-model="pledgeData.period"
        type="datetimerange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
      >
      </el-date-picker>
      <el-button type="primary" @click="getPledgeNatOrNrNat('pledge', pledgeData.period)" round>get pledge nat</el-button>
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
          pledgeList: [],
          pledgeNat: 0,
          show: false,
          topic: 'chain.contract.NATToken'
        },
        nrData: {
          period: [],
          nrList: [],
          nrNat: 0,
          show: false,
          topic: 'chain.contract.NATToken'
        },
        voteData: {
          period: [],
          voteList: [],
          rewardNat: 0,
          burnNat: 0,
          show: false,
          topic: 'chain.contract.vote'
        },
        AddressData: {
          address: '',
          period: [],
          pledge: {
            nat: 0
          },
          nr: {
            nat: 0
          },
          vote: {
            rewardNat: 0,
            usedNat: 0,
            burnNat: 0
          },
          str: '',
          show: false
        }
      }
    },
    mounted() {
      this.get(this.getTxUrl, this.pledgeAndNrAddress)
      this.get(this.getTxUrl, this.voteAddress)
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
                  let json = JSON.parse(list[j].data);
                  if (json.Function === 'triggerNR') {
                    let item = {
                      hash: list[j].hash,
                      timestamp: list[j].timestamp
                    }
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
          list = this.pledgeData.pledgeList
          topic = this.pledgeData.topic
          this.pledgeData.pledgeNat = 0
          for (let m = 0; m < list.length; m++) {
            if (list[m].timestamp > startTimestamp && list[m].timestamp < endTimestamp) {
              periodList.push(list[m])
            }
          }
        }
        if (tag === 'nr') {
          list = this.nrData.nrList
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
                    this.pledgeData.pledgeNat += parseFloat(data[k].value)
                    if (address.length > 0 && address === data[k].addr) {
                      this.AddressData.pledge.nat += parseFloat(data[k].value)
                    }
                  }
                  if (tag === 'nr') {
                    this.nrData.nrNat += parseFloat(data[k].value)
                    if (address.length > 0 && address === data[k].addr) {
                      this.AddressData.nr.nat += parseFloat(data[k].value)
                    }
                  }
                }
              }
            }
          });
        }
        this.sleep(3000).then(() => {
          if (tag === 'nr') {
            this.nrData.str = '';
            this.nrData.show = false
            this.nrData.str = 'NR计算周期：' + this.dateFormat(startTimestamp) + '-' + this.dateFormat(endTimestamp) + '\n\\'
              + ', 通过NR发放的NAT数量: ' + parseInt(this.nrData.nrNat / 1000000000000000000)
            this.nrData.show = true
          }
          if (tag === 'pledge') {
            this.pledgeData.str = ''
            this.pledgeData.show = false
            this.pledgeData.str = '质押周期：' + this.dateFormat(startTimestamp - 3600 * 1000 * 24 * 7) + '-' + this.dateFormat(endTimestamp - 3600 * 1000 * 24 * 7)
              + ', 通过该周期质押进行发放的NAT数量共有: ' + parseInt(this.pledgeData.pledgeNat / 1000000000000000000);
            this.pledgeData.show = true
          }
          if (address.length > 0) {
            this.AddressData.str = ''
            this.AddressData.show = false
            this.AddressData.str = '地址: ' + address + ', 在'
          }
        })
      },
      getVoteNat(period, address = '') {
        console.log('address: ', address.length)
        if (period.length !== 2) {
          this.$notify.error({
            title: 'failed',
            message: 'select time first',
            type: 'error',
            duration: 1000
          })
          return
        }
        let list = this.voteData.voteList
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
        for (let j = 0; j < periodList.length; j++) {
          axios.post(this.getEventByHashUrl,
            JSON.stringify({hash: periodList[j].hash}),
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }).then(response => {
            let events = response.data.result.events
            for (let k = 0; k < events.length; k++) {
              if (events[k].topic === topic) {
                let result = JSON.parse(events[k].data)
                this.voteData.rewardNat += parseFloat(result.reward)
                let voteBurnNatList = result.data
                this.voteData.burnNat += parseFloat(voteBurnNatList[1].nat)
                if (address.length > 0 && address === voteBurnNatList[1].addr) {
                  this.AddressData.vote.burnNat += parseFloat(voteBurnNatList[1].nat)
                  this.AddressData.vote.rewardNat += parseFloat(result.reward)
                }
              }
            }
          })
        }
        this.sleep(3000).then(() => {
          this.voteData.str = ''
          this.voteData.show = false
          this.voteData.str = '投票周期: ' + this.dateFormat(startTimestamp) + '-' + this.dateFormat(endTimestamp)
            + ', 本周已投出的NAT中获得激励的数量: ' + parseInt(this.voteData.rewardNat / 10)
          this.voteData.show = true
        })

      },
      getAddressData() {
        if (this.AddressData.period.length !== 2) {
          this.$message({
            type: 'error',
            message: 'please select time first',
            duration: 1000
          })
          return
        }
        let period = this.AddressData.period
        let address = this.AddressData.address
        this.getPledgeNatOrNrNat('pledge', period, address)
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
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .vote-data {

  }
</style>
