<template>
    <div>
        <b-breadcrumb :items="items"/>
        <b-card v-if="account == null"
                :title="`账户 ${account_name}`"
                tag="article"
                style=""
                class="mb-2">
            <p class="card-text">
                查无此账户
            </p>
        </b-card>
        <b-card v-if="account != null"
                :title="`账户 ${account_name}`"
                tag="article"
                style=""
                class="mb-2">
            <p class="card-text">
                <b-row>
                    <b-col xs="12" sm="12" md="3" lg="4" class="account-text">
                        <b-row>
                            <b-col xs="12" sm="12">
                                <div style="margin-bottom: 4px;">主币余额</div>
                                <div style="font-size: 20px;">{{account.core_liquid_balance == undefined ? '0.0000' :
                                    account.core_liquid_balance}}
                                </div>
                            </b-col>
                        </b-row>
                    </b-col>
                    <b-col xs="12" sm="12" md="9" lg="8" class="account-text">
                        <b-row>
                            <b-col xs="12" sm="4">
                                <div style="margin-bottom: 8px;">RAM</div>
                                <b-progress :value="((account.ram_quota - account.ram_usage) / account.ram_quota * 100)"
                                            height="0.8rem" style="margin-bottom: 6px;"></b-progress>
                                <div v-if="account.ram_quota == -1" style="margin-bottom: 8px;">-</div>
                                <div v-else style="margin-bottom: 8px;">剩余 {{getByteSize(account.ram_quota -
                                    account.ram_usage)}}/{{getByteSize(account.ram_quota)}}
                                </div>
                            </b-col>
                            <b-col xs="12" sm="4">
                                <div style="margin-bottom: 8px;">CPU</div>
                                <b-progress :value="(account.cpu_limit.available / account.cpu_limit.max * 100)"
                                            height="0.8rem" style="margin-bottom: 6px;"></b-progress>
                                <div v-if="account.cpu_limit.max == -1" style="margin-bottom: 8px;">-</div>
                                <div v-else style="margin-bottom: 8px;">剩余
                                    {{getTimeSize(account.cpu_limit.available)}}/{{getTimeSize(account.cpu_limit.max)}}
                                </div>
                            </b-col>
                            <b-col xs="12" sm="4">
                                <div style="margin-bottom: 8px;">NET</div>
                                <b-progress :value="(account.net_limit.available / account.net_limit.max * 100)"
                                            height="0.8rem" style="margin-bottom: 6px;"></b-progress>
                                <div v-if="account.net_limit.max == -1" style="margin-bottom: 8px;">-</div>
                                <div v-else style="margin-bottom: 8px;">剩余
                                    {{getByteSize(account.net_limit.available)}}/{{getByteSize(account.net_limit.max)}}
                                </div>
                            </b-col>
                        </b-row>
                    </b-col>
                </b-row>
            </p>
        </b-card>
        <b-card v-if="account != null"
                :title="`代币资产`"
                tag="article"
                style="margin-top: 16px;"
                class="mb-2">
            <p class="card-text" style="padding-top: 12px;">
                <b-row>
                    <b-col xs="12" sm="6" md="4" lg="3" v-for="(tk, index) in accountTokenList" :key="'tk_' + index">
                        <b-card no-body style="padding: 12px; margin-bottom: 12px;">
                            <div style="display: flex; justify-content: space-between; align-items: center;">
                                <span style="font-size: 20px; color: #007bff;">{{tk.symbol}}</span>
                                <span style="font-size: 15px; color: #6c757d;">{{tk.code}}</span>
                            </div>
                            <div style="display: flex; justify-content: flex-start; align-items: center;">
                                <span style="font-size: 20px; font-weight: 500;">{{(tk.balance + '').split(' ')[0]}}</span>
                                <span style="font-size: 15px; color: #6c757d; margin-left: 10px;">{{tk.symbol}}</span>
                            </div>
                        </b-card>
                    </b-col>
                </b-row>
            </p>
        </b-card>
        <b-card v-if="account != null"
                no-body style="margin-top: 16px; font-size: 14px;">
            <b-tabs card>
                <b-tab :title="`Actions(${actions.length})`" active>
                    <b-card no-body class="account-tr d-none d-md-block d-lg-block">
                        <b-row>
                            <b-col xs="12" sm="12" md="1" style="display: flex; align-items: center;">
                                交易
                            </b-col>
                            <b-col xs="12" sm="12" md="2" style="display: flex; align-items: center;">
                                交易时间
                            </b-col>
                            <b-col xs="12" sm="12" md="3" style="display: flex; align-items: center;">
                                合约和方法
                            </b-col>
                            <b-col xs="12" sm="12" md="5" style="display: flex; align-items: center;">
                                交易数据
                            </b-col>
                        </b-row>
                    </b-card>
                    <b-card no-body v-for="(ac, index) in actions" :key="'ac_' + index"
                            class="account-tr">
                        <b-row>
                            <b-col xs="12" sm="12" md="1" style="display: flex; align-items: center;">
                                <div class="sak-text">
                                    <span style="cursor: pointer; color: #007bff;"
                                          @click="GoTransaction(ac.action_trace.trx_id)">{{ac.action_trace.trx_id}}
                                    </span>
                                </div>
                            </b-col>
                            <b-col xs="12" sm="12" md="2" style="display: flex; align-items: center;">
                                {{GetMoment(ac.block_time)}}
                            </b-col>
                            <b-col xs="12" sm="12" md="3" style="display: flex; align-items: center;">
                                <span style="cursor: pointer; color: #007bff;"
                                      @click="GoAccount(ac.action_trace.act.account)">{{ac.action_trace.act.account}}</span>::{{ac.action_trace.act.name}}
                            </b-col>
                            <b-col xs="12" sm="12" md="5" style="display: flex; align-items: center;">
                                        <pre style="margin-bottom: 0;">
{{JSON.stringify(ac.action_trace.act.data, null, 4)}}
                                        </pre>
                            </b-col>
                        </b-row>
                    </b-card>
                </b-tab>
            </b-tabs>
        </b-card>
    </div>
</template>

<script>
    import * as moment from 'moment'

    export default {
        name: 'Account',
        data() {
            return {
                account_name: '',
                items: [
                    {
                        text: '首页',
                        to: {name: 'Home'}
                    }
                ],
                account: null,
                accountTokenList: [],
                actions: []
            }
        },
        created() {
            let self = this
            self.account_name = self.$route.params.account_name
            self.items.push(
                {
                    text: `账户 ${self.account_name}`
                }
            )
            if (self.account_name != '') {
                self.GetInfo()
            }
        },
        watch: {
            // 如果路由有变化，会再次执行该方法
            '$route': function () {
                this.account_name = this.$route.params.account_name
                this.account = null
                this.items[1].text = `账户 ${this.account_name}`
                this.GetInfo()
            }
        },
        methods: {
            GetMoment: function (date) {
                return moment(date).utcOffset(960).format('YYYY-MM-DD HH:mm:ss')
            },
            GetInfo() {
                let self = this
                let p1 = self.$parent.getAccount(self.account_name).then(r => {
                    // console.log(r)
                    self.account = r
                }).catch(e => {
                    console.log(e)
                })
                let p2 = self.$parent.getBalances(self.account_name).then(r => {
                    // console.log(r)
                    self.accountTokenList = r
                }).catch(e => {
                    console.log(e)
                })
                let p3 = self.$parent.getActions(self.account_name).then(r => {
                    // console.log(r)
                    let res = r.actions.reverse()
                    // console.log(res)
                    self.actions = res
                }).catch(e => {
                    console.log(e)
                })
                Promise.all([p1, p2, p3]).then(() => {
                    self.$parent.StopLoading()
                }).catch(() => {
                    self.$parent.StopLoading()
                })
            },
            getByteSize: function (net) {
                let len = 2
                let s = ''
                if (net < 1024) {
                    s = net.toFixed(len).toString() + 'B'
                } else {
                    net /= 1024
                    if (net < 1024) {
                        s = net.toFixed(len).toString() + 'KB'
                    } else {
                        net /= 1024
                        if (net < 1024) {
                            s = net.toFixed(len).toString() + 'MB'
                        } else {
                            net /= 1024
                            if (net < 1024) {
                                s = net.toFixed(len).toString() + 'GB'
                            } else {
                                net /= 1024
                                s = net.toFixed(len).toString() + 'TB'
                            }
                        }
                    }
                }
                return s
            },
            getTimeSize: function (net) {
                let len = 2
                let s = ''
                if (net < 1000) {
                    s = net.toFixed(len).toString() + 'ns'
                } else {
                    net /= 1000
                    if (net < 1000) {
                        s = net.toFixed(len).toString() + 'ms'
                    } else {
                        net /= 1000
                        if (net < 60) {
                            s = net.toFixed(len).toString() + 'sec'
                        } else {
                            net /= 60
                            if (net < 60) {
                                s = net.toFixed(len).toString() + 'min'
                            } else {
                                net /= 60
                                s = net.toFixed(len).toString() + 'hr'
                            }
                        }
                    }
                }
                return s
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
            GoHome() {
                this.$router.push({path: `/`})
            }
        }
    }
</script>

<style scoped>
    .account-text {
        margin: 3px 0 3px 0;
        font-size: 15px;
    }

    .account-tr {
        margin-bottom: 10px;
        padding: 10px 15px 10px 15px;
        overflow: hidden;
    }

    .account-tr:last-child {
        margin-bottom: 0;
    }
</style>
