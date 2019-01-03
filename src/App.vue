<template>
    <div>
        <div style="width: 100%; height: auto; background-color: #6c757d;">
            <b-navbar toggleable="md" type="dark" variant="secondary" @keyup.enter="GoSearch"
                      style="max-width: 1000px; margin: 0 auto;">
                <b-navbar-brand style="letter-spacing: 2px; font-weight: 600; cursor: pointer;" @click="GoHome">
                    {{title}}
                </b-navbar-brand>
                <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
                <b-collapse is-nav id="nav_collapse">
                    <b-navbar-nav>
                        <b-nav-item>&nbsp;</b-nav-item>
                    </b-navbar-nav>
                    <b-navbar-nav class="ml-auto">
                        <b-nav-item-dropdown v-show="false" :text="netConfig == null ? '' : netConfig.netName" right
                                             style="width: 130px; text-align: left; margin-right: 15px;">
                            <b-dropdown-item v-for="nc in netConfigList" :key="nc.netId" @click="ChangeNet(nc.netId)">
                                {{nc.netName}}
                            </b-dropdown-item>
                        </b-nav-item-dropdown>
                        <b-nav-form>
                            <b-form-input type="text" placeholder="区块 / 交易 / 账户"
                                          v-model="keyword"
                                          style="min-width: 280px; max-width: 360px; margin-right: 10px;"/>
                            <b-button type="button" style="margin-right: 0;" @click="GoSearch">查询</b-button>
                        </b-nav-form>
                    </b-navbar-nav>
                </b-collapse>
            </b-navbar>
        </div>
        <div style="max-width: 1000px; margin: 8px auto; padding: 8px;">
            <router-view/>
        </div>
        <div v-show="isLoading"
             style="position: fixed; top: 0; bottom: 0; left: 0; right: 0; background-color: rgba(255,255,255,0.80); z-index: 999; display: flex; justify-content: center; align-items: center;">
            <img src="loading.png" alt="" class="sak-loading"/>
            <div style="margin-left: 28px; font-size: 24px; font-weight: 500;">查询中，请稍等</div>
        </div>
    </div>
</template>

<script>
    /* eslint-disable no-undef */

    let EosApi = require('eosjs-api') // 引入只读EOS

    export default {
        name: 'App',
        data() {
            return {
                title: title,
                netConfigList: netConfigList,
                netConfig: null,
                keyword: '',
                eosApi: null,
                isLoading: false
            }
        },
        created() {
            let self = this
            self.netConfig = self.netConfigList[0]
            self.eosApi = EosApi({
                httpEndpoint: self.netConfig.httpEndpoint
            })
        },
        watch: {},
        methods: {
            ChangeNet(netId) {
                let self = this
                for (let i in self.netConfigList) {
                    let tmp = self.netConfigList[i]
                    if (tmp.netId == netId) {
                        self.netConfig = tmp
                        self.eosApi = EosApi({
                            httpEndpoint: self.netConfig.httpEndpoint
                        })
                        self.$router.replace({path: `/Home/${self.netConfig.netName}`})
                    }
                }
            },
            GoSearch() {
                let self = this
                let kw = self.keyword
                if (kw == '') {
                    console.log('no')
                } else if (!isNaN(kw)) {
                    self.GoBlock(kw)
                    self.keyword = ''
                } else if (kw.length == 64) {
                    self.GoTransaction(kw)
                    self.keyword = ''
                } else {
                    self.GoAccount(kw)
                    self.keyword = ''
                }
            },
            GoHome() {
                this.$router.push({path: `/`})
            },
            GoBlock(block_num) {
                this.$router.push({path: `/Block/${block_num}`})
            },
            GoTransaction(trx_id) {
                this.$router.push({path: `/Transaction/${trx_id}`})
            },
            GoAccount(account) {
                this.$router.push({path: `/Account/${account}`})
            },
            StartLoading() {
                this.isLoading = true
            },
            StopLoading() {
                this.isLoading = false
            },
            //======================== db =========================
            getNewTrx() {
                let self = this
                self.StartLoading()
                if (self.netConfig.dbAddress != '') {
                    return self.$http.get(`${self.netConfig.dbAddress}/eosSak/db/GetNewTransactionsNew`)
                } else {
                    return Promise.resolve()
                }
            },
            //======================== api =========================
            getInfo() {
                let self = this
                self.StartLoading()
                return self.eosApi.getInfo({})
            },
            getBlock(bid) {
                let self = this
                self.StartLoading()
                return self.eosApi.getBlock(bid)
            },
            getTransaction(tid) {
                let self = this
                self.StartLoading()
                return self.eosApi.getTransaction(tid)
            },
            getAccount(acc) {
                let self = this
                self.StartLoading()
                return self.eosApi.getAccount(acc)
            },
            getBalances(acc) {
                let self = this
                let ps = []
                for (let i in self.netConfig.tokenList) {
                    let obj = self.netConfig.tokenList[i]
                    let newObj = {
                        code: obj.code,
                        symbol: obj.symbol,
                        balance: 0
                    }
                    let p = self.eosApi.getCurrencyBalance(newObj.code, acc, newObj.symbol).then(r => {
                        if (r.length == 1) {
                            newObj.balance = r[0]
                        } else {
                            newObj.balance = 0
                        }
                        return newObj
                    }).catch(e => {
                        console.log(e)
                        newObj.balance = 0
                        return newObj
                    })
                    ps.push(p)
                }
                self.StartLoading()
                return Promise.all(ps)
            },
            getActions(acc) {
                let self = this
                self.StartLoading()
                return self.eosApi.getActions(acc, -1, -30)
            }
        }
    }
</script>

<style>
    .sak-text {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .sak-loading {
        -webkit-animation: rotateImg 1s linear infinite;
        width: 48px;
        height: 48px;
        vertical-align: middle;
    }

    @keyframes rotateImg {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    @-webkit-keyframes rotateImg {
        0% {
            -webkit-transform: rotate(0deg);
        }
        100% {
            -webkit-transform: rotate(360deg);
        }
    }
</style>
