<template>
    <div>
        <b-breadcrumb :items="items" class="sak-text"/>
        <b-card v-if="trx == null"
                :title="`交易 ${trx_id}`"
                tag="article"
                style=""
                class="mb-2">
            <p class="card-text">
                查无此交易
            </p>
        </b-card>
        <b-card v-if="trx != null"
                :title="`交易 ${trx_id}`"
                tag="article"
                style=""
                class="mb-2">
            <p class="card-text">
                <!--<b-row>-->
                <!--<b-col xs="12" class="transaction-text">-->
                <!--状态：-->
                <!--</b-col>-->
                <!--</b-row>-->
                <b-row>
                    <b-col xs="12" sm="6" class="transaction-text">
                        区块时间：{{GetMoment(trx.block_time)}}
                    </b-col>
                    <b-col xs="12" sm="6" class="transaction-text" v-if="trx.trx.trx != undefined">
                        过期时间：{{GetMoment(trx.trx.trx.expiration)}}
                    </b-col>
                </b-row>
                <b-row>
                    <b-col xs="12" sm="6" class="transaction-text">
                        <span>状态：</span>
                        <b-badge variant="success" style="font-size: 100%; color: white;"
                                 v-if="trx.trx.receipt.status == 'executed'">
                            已执行
                        </b-badge>
                        <b-badge variant="warning" style="font-size: 100%; color: white;" v-else>未执行</b-badge>
                    </b-col>
                    <b-col xs="12" sm="6" class="transaction-text">
                        所在区块：<span style="cursor: pointer; color: #007bff;" @click="GoBlock(trx.block_num)">#{{trx.block_num}}</span>
                    </b-col>
                </b-row>
                <!--<b-row>-->
                <!--<b-col xs="12" class="transaction-text-text">-->
                <!--交易Hash：{{trx.id}}-->
                <!--</b-col>-->
                <!--</b-row>-->
            </p>
        </b-card>
        <b-card v-if="trx != null"
                no-body style="margin-top: 16px; font-size: 14px;">
            <b-tabs card>
                <b-tab :title="`Actions(${trx.trx.trx.actions.length})`" active v-if="trx.trx.trx != undefined">
                    <b-card no-body class="transaction-tr d-none d-md-block d-lg-block">
                        <b-row>
                            <b-col xs="12" sm="12" md="3" style="display: flex; align-items: center;">
                                合约和方法
                            </b-col>
                            <b-col xs="12" sm="12" md="9" style="display: flex; align-items: center;">
                                交易数据
                            </b-col>
                        </b-row>
                    </b-card>
                    <b-card no-body v-for="(ac, index) in trx.trx.trx.actions" :key="'ac_' + index"
                            class="transaction-tr">
                        <b-row>
                            <b-col xs="12" sm="3" style="display: flex; align-items: center;">
                                <span style="cursor: pointer; color: #007bff;" @click="GoAccount(ac.account)">{{ac.account}}</span>::{{ac.name}}
                            </b-col>
                            <b-col xs="12" sm="9"
                                   style="display: flex; justify-content: flex-start; align-items: flex-start; flex-direction: column;">
                                <template v-if="ac.name == 'transfer' || ac.name == 'issue'">
                                    <div style="width: 100%; height: 24px; display: flex; align-items: center;">
                                        <b-badge variant="secondary" style="margin-right: 10px;"
                                                 v-if="ac.name == 'transfer'">转账方向
                                        </b-badge>
                                        <b-badge variant="secondary" style="margin-right: 10px;"
                                                 v-if="ac.name == 'issue'">发行代币
                                        </b-badge>
                                        <span style="cursor: pointer; color: #007bff;"
                                              @click="GoAccount(ac.data.from)">
                                            {{ac.data.from}}
                                        </span>
                                        <span style="margin-left: 8px; margin-right: 8px;">
                                            ->
                                        </span>
                                        <span style="cursor: pointer; color: #007bff;"
                                              @click="GoAccount(ac.data.to)">
                                            {{ac.data.to}}
                                        </span>
                                    </div>
                                    <div style="width: 100%; height: 24px; display: flex; align-items: center;">
                                        <b-badge variant="secondary" style="margin-right: 10px;"
                                                 v-if="ac.name == 'transfer'">转账金额
                                        </b-badge>
                                        <b-badge variant="secondary" style="margin-right: 10px;"
                                                 v-if="ac.name == 'issue'">发行金额
                                        </b-badge>
                                        {{ac.data.quantity}}
                                    </div>
                                    <div style="width: 100%; min-height: 24px; display: flex; align-items: center; word-break: break-all;">
                                        <b-badge variant="secondary" style="margin-right: 19px;">MEMO</b-badge>
                                        {{ac.data.memo}}
                                    </div>
                                </template>
                                <pre v-else style="margin-bottom: 0;">
{{JSON.stringify(ac.data, null, 4)}}
                                </pre>
                            </b-col>
                        </b-row>
                    </b-card>
                </b-tab>
                <b-tab title="JSON视图">
                    <pre style="margin-bottom: 0;">
{{JSON.stringify(trx, null, 4)}}
                    </pre>
                </b-tab>
            </b-tabs>
        </b-card>
    </div>
</template>

<script>
    import * as moment from 'moment'

    export default {
        name: 'Transaction',
        data() {
            return {
                trx_id: '',
                items: [
                    {
                        text: '首页',
                        to: {name: 'Home'}
                    }
                ],
                trx: null
            }
        },
        created() {
            let self = this
            self.trx_id = self.$route.params.trx_id
            self.items.push(
                {
                    text: `交易 ${self.trx_id}`
                }
            )
            if (self.trx_id != '') {
                self.GetInfo()
            }
        },
        watch: {
            // 如果路由有变化，会再次执行该方法
            '$route': function () {
                this.trx_id = this.$route.params.trx_id
                this.trx = null
                this.items[1].text = `交易 ${this.trx_id}`
                this.GetInfo()
            }
        },
        methods: {
            GetMoment: function (date) {
                return moment(date).utcOffset(960).format('YYYY-MM-DD HH:mm:ss')
            },
            GetInfo() {
                let self = this
                self.$parent.getTransaction(self.trx_id).then(r => {
                    // console.log(r)
                    self.trx = r
                    self.$parent.StopLoading()
                }).catch(e => {
                    console.log(e)
                    self.$parent.StopLoading()
                })
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
    .transaction-text {
        margin: 3px 0 3px 0;
    }

    .transaction-tr {
        margin-bottom: 10px;
        padding: 10px 15px 10px 15px;
        overflow: hidden;
    }

    .transaction-tr:last-child {
        margin-bottom: 0;
    }
</style>
