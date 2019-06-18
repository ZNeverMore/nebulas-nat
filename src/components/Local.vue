<template>
  <div style="margin-top: 50px">
    <el-date-picker
      v-model="pledgeData.period"
      type="datetimerange"
      range-separator="至"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
    >
    </el-date-picker>
    <el-button type="primary" @click="savePledgeTxList" round>save pledge tx</el-button>
    <el-button type="primary" @click="saveNrTxList" round>save nr tx</el-button>
    <el-button type="primary" @click="saveVoteTxList" round>save vote tx</el-button>
    <el-button type="primary" @click="test" round>test</el-button>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    name: "Local",
    data() {
      return {
        pledgeAndNrAddress: 'n1EoNsJNXG1tN3z9rvjwPKoBXbJMqAjmESC',
        voteAddress: 'n1pADU7jnrvpPzcWusGkaizZoWgUywMRGMY',
        getTxUrl: 'https://explorer-backend.nebulas.io/api/tx/',
        getEventByHashUrl: 'https://mainnet.nebulas.io/v1/user/getEventsByHash',
        pledgeData: {
          pledgeTxList: [],
          pledgeTxEventsList: [],
          topic: 'chain.contract.NATToken',
        },
        nrData: {
          nrTxList: [],
          nrTxEventsList: [],
          topic: 'chain.contract.NATToken',
        },
        voteData: {
          voteTxList: [],
          voteTxEventsList: [],
          topic: 'chain.contract.vote',
        }
      }
    },
    mounted() {
      this.get(this.getTxUrl, this.pledgeAndNrAddress, 'pledgeAndNr')
      this.get(this.getTxUrl, this.voteAddress, 'vote')
    },
    methods: {
      test() {
        let str = '64221f3d8ef895e9ecc3eec4ccb969e0c905f164af3f1a35f40922091a3372ad'
        axios.post(this.getEventByHashUrl,
          JSON.stringify({hash: str}),
          {
            headers: {
              'Content-Type': 'application/json'
            }
          }).then(response => {
            console.log(response)
        })
        // let item = {
        //   hash: 'test',
        //   timestamp: 1234567891,
        //   usedNat: '100'
        // }
        // axios.post('/api/vote/insert/single', item).then(response => {
        //   if (response.data.code === 200) {
        //     this.getVoteEventsByHash(hash)
        //   } else {
        //     console.log('insert vote tx error response', response.data)
        //   }
        // })
      },
      get(url, address, tag) {
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
                  let hash = list[j].hash
                  let json = JSON.parse(list[j].data)
                  if (json.Function === 'triggerNR') {
                    let item = {
                      hash: hash,
                      timestamp: list[j].timestamp
                    }
                    this.nrData.nrTxList.push(item)
                    axios.post('/api/nr/insert/single', item).then(response => {
                      if (response.data.code !== 200) {
                        console.log('insert nr tx error response', response.data)
                      }
                    })

                  }
                  if (json.function === 'triggerPledge' || json.Function === 'triggerPledge') {
                    let item = {
                      hash: hash,
                      timestamp: list[j].timestamp
                    }
                    this.pledgeData.pledgeTxList.push(item)
                    axios.post('/api/pledge/insert/single', item).then(response => {
                      if (response.data.code !== 200) {
                        console.log('insert pledge tx error response', response.data)
                      }
                    })

                  }
                  if (json.Function === 'vote') {
                    let argsList = JSON.parse(json.Args)
                    let voteUsedNat = parseInt(argsList[argsList.length - 1] / 1000000000000000000)
                    let item = {
                      hash: hash,
                      timestamp: list[j].timestamp,
                      usedNat: voteUsedNat
                    }
                    this.voteData.voteTxList.push(item)
                    axios.post('/api/vote/insert/single', item).then(response => {
                      if (response.data.code !== 200) {
                        console.log('insert vote tx error response', response.data)
                      }
                    })
                  }
                }
                if (i === totalPage && j === list.length - 1) {
                  console.log('get complete')
                }
              }
              this.sleep(5000).then(() => {
                if (tag === 'pledgeAndNr') {
                  let pledgeTxList = this.pledgeData.pledgeTxList
                  for (let i = 0; i < pledgeTxList.length; i++) {
                    this.getPledgeEventsByHash(pledgeTxList[i])
                  }
                }
              })
              this.sleep(10000).then(() => {
                if (tag === 'pledgeAndNr') {
                  let nrTxList = this.nrData.nrTxList;
                  for (let m = 0; m < nrTxList.length; m++) {
                    this.getNrEventsByHash(nrTxList[i])
                  }
                }
              })
              this.sleep(15000).then(() => {
                if (tag === 'vote') {
                  let voteTxList = this.voteData.voteTxList
                  for (let n = 0; n < voteTxList.length; n++) {
                    this.getVoteEventsByHash(voteTxList[n])
                  }
                }
              })
            })
          }
        })
      },
      savePledgeTxList() {
        if (this.pledgeData.pledgeTxList.length === 0) {
          return
        }
        let params = {
          pledgeTxList: this.pledgeData.pledgeTxList
        }
        axios.post('/api/pledge/insert', params).then(response => {
          console.log(response)
        })
      },
      getPledgeEventsByHash(hash) {
        let topic = this.pledgeData.topic
        axios.post(this.getEventByHashUrl,
          JSON.stringify({hash: hash}),
          {
            headers: {
              'Content-Type': 'application/json'
            }
          }).then(response => {
          let events = response.data.result.events
          for (let i = 0; i < events.length; i++) {
            if (events[i].topic === topic) {
              let result = JSON.parse(events[i].data)
              let list = result.Produce.data
              for (let j = 0; j < list.length; j++) {
                let item = {
                  addr: list[j].addr,
                  value: list[j].value,
                  hash: hash
                }
                this.pledgeData.pledgeTxEventsList.push(item)
              }
            }
          }
          this.sleep(10000).then(() => {
            this.savePledgeTxEventsListByHash()
          })
        })
      },
      savePledgeTxEventsListByHash() {
        let list = this.pledgeData.pledgeTxEventsList
        for (let i = 0; i < list.length; i++) {
          axios.post('/api/pledge/insert/nat', list[i]).then(response => {
            if (response.data.code !== 200) {
              console.log('insert pledge nat by hash error: ',response.data)
            }
          })
        }
      },
      saveNrTxList() {
        if (this.nrData.nrTxList.length === 0) {
          return
        }
        let params = {
          nrTxList: this.nrData.nrTxList
        }
        axios.post('/api/nr/insert', params).then(response => {
          console.log(response)
        })
      },
      getNrEventsByHash(hash) {
        let topic = this.nrData.topic
        axios.post(this.getEventByHashUrl,
          JSON.stringify({hash: hash}),
          {
            headers: {
              'Content-Type': 'application/json'
            }
          }).then(response => {
          let events = response.data.result.events
          for (let i = 0; i < events.length; i++) {
            if (events[i].topic === topic) {
              let result = JSON.parse(events[i].data)
              let list = result.Produce.data
              for (let j = 0; j < list.length; j++) {
                let item = {
                  addr: list[j].addr,
                  value: list[j].value,
                  hash: hash
                }
                this.nrData.nrTxEventsList.push(item)
              }
            }
          }
          this.sleep(10000).then(() => {
            this.saveNrTxEventsListByHash()
          })
        })
      },
      saveNrTxEventsListByHash() {
        let list = this.nrData.nrTxEventsList
        for (let i = 0; i < list.length; i++) {
          axios.post('/api/nr/insert/nat', list[i]).then(response => {
            if (response.data.code !== 200) {
              console.log('insert nr nat by hash error: ', response.data)
            }
          })
        }
      },
      saveVoteTxList() {
        if (this.voteData.voteTxList.length === 0) {
          return
        }
        let params = {
          voteTxList: this.voteData.voteTxList
        }
        axios.post('/api/vote/insert/hash', params).then(response => {
          console.log(response)
        })
      },
      getVoteEventsByHash(hash) {
        let topic = this.voteData.topic
        axios.post(this.getEventByHashUrl,
          JSON.stringify({hash: hash}),
          {
            headers: {
              'Content-Type': 'application/json'
            }
          }).then(response => {
          let events = response.data.result.events
          for (let i = 0; i < events.length; i++) {
            if (events[i].topic === topic) {
              let result = JSON.parse(events[i].data)
              let voteBurnNatList = result.data
              let item = {
                reward: result.reward,
                addr: voteBurnNatList[1].addr,
                burnNat: voteBurnNatList[1].nat,
                hash: hash
              }
              this.voteData.voteTxEventsList.push(item)
            }
          }
          this.sleep(10000).then(() => {
            this.saveVoteTxEventsListByHash()
          })
        })
      },
      saveVoteTxEventsListByHash() {
        let list = this.voteData.voteTxEventsList
        for (let i = 0; i < list.length; i++) {
          axios.post('/api/vote/insert/reward', list[i]).then(response => {
            if (response.data.code !== 200) {
              console.log('insert vote reward error: ', response.data)
            }
          })
        }
      },
      sleep(milliseconds) {
        return new Promise(resolve => setTimeout(resolve, milliseconds))
      },
    }
  }
</script>

<style scoped>

</style>
