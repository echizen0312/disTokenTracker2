<template>
    <div>
        <b-breadcrumb :items="items"/>
        <b-card v-if="trx != null"
                :title="`交易 ${trx_id}`"
                tag="article"
                style=""
                class="mb-2">
            <p class="card-text">
                <b-row>
                    <b-col xs="12" class="transaction-text">
                        状态：
                    </b-col>
                </b-row>
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
                no-body style="margin-top: 16px; font-size: 15px;">
            <b-tabs card>
                <b-tab :title="`Actions(${trx.trx.trx.actions.length})`" active v-if="trx.trx.trx != undefined">
                    <b-card no-body v-for="(ac, index) in trx.trx.trx.actions" :key="'ac_' + index"
                            class="transaction-tr">
                        <b-row>
                            <b-col xs="12" sm="3" style="display: flex; align-items: center;">
                                <span style="cursor: pointer; color: #007bff;" @click="GoAccount(ac.account)">{{ac.account}}</span>::{{ac.name}}
                            </b-col>
                            <b-col xs="12" sm="9" style="display: flex; align-items: center;">
                                        <pre style="margin-bottom: 0;">
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
                }).catch(e => {
                    console.log(e)
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
