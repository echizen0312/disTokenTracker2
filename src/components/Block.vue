<template>
    <div>
        <b-breadcrumb :items="items"/>
        <b-card v-if="block != null"
                :title="`区块 # ${block_num}`"
                tag="article"
                style=""
                class="mb-2">
            <p class="card-text">
                <b-row>
                    <b-col xs="12" class="block-text">
                        状态：
                    </b-col>
                </b-row>
                <b-row>
                    <b-col xs="12" sm="6" class="block-text">
                        出块时间：{{GetMoment(block.timestamp)}}
                    </b-col>
                    <b-col xs="12" sm="6" class="block-text">
                        出块节点：{{block.producer}}
                    </b-col>
                </b-row>
                <b-row>
                    <b-col xs="12" sm="6" class="block-text">
                        前一区块：<span style="cursor: pointer; color: #007bff;" @click="GoBlock(block_num - 1)">#{{block_num - 1}}</span>
                    </b-col>
                    <b-col xs="12" sm="6" class="block-text">
                        后一区块：<span style="cursor: pointer; color: #007bff;" @click="GoBlock(block_num + 1)">#{{block_num + 1}}</span>
                    </b-col>
                </b-row>
                <b-row>
                    <b-col xs="12" class="block-text">
                        区块Hash：{{block.id}}
                    </b-col>
                </b-row>
            </p>
        </b-card>
        <b-card v-if="block != null"
                no-body style="margin-top: 16px; font-size: 15px;">
            <b-tabs card>
                <b-tab :title="`交易(${block.transactions.length})`" active>
                    <b-card no-body v-for="(tr, index) in block.transactions" :key="'tr_' + index" class="block-tr">
                        <b-row>
                            <b-col xs="12" sm="2" style="display: flex; align-items: center;">
                                <span class="sak-text" style="cursor: pointer; color: #007bff;"
                                      @click="GoTransaction(typeof tr.trx == 'string' ? tr.trx : tr.trx.id)">{{typeof tr.trx == 'string' ? tr.trx : tr.trx.id}}</span>
                            </b-col>
                            <b-col xs="12" sm="10" class="">
                                <b-row v-if="typeof tr.trx == 'string'">
                                    <b-col xs="12" sm="12">
                                        <span>内联交易</span>
                                    </b-col>
                                </b-row>
                                <b-row v-else v-for="(ac, indexAc) in tr.trx.transaction.actions"
                                       :key="index + 'ac_' + indexAc">
                                    <b-col xs="12" sm="4" style="display: flex; align-items: center;">
                                        <span style="cursor: pointer; color: #007bff;" @click="GoAccount(ac.account)">{{ac.account}}</span>::{{ac.name}}
                                    </b-col>
                                    <b-col xs="12" sm="8" style="display: flex; align-items: center;">
                                        <pre style="margin-bottom: 0;">
{{JSON.stringify(ac.data, null, 4)}}
                                        </pre>
                                    </b-col>
                                </b-row>
                            </b-col>
                        </b-row>
                    </b-card>
                </b-tab>
                <b-tab title="JSON视图">
                    <pre style="margin-bottom: 0;">
{{JSON.stringify(block, null, 4)}}
                    </pre>
                </b-tab>
            </b-tabs>
        </b-card>
    </div>
</template>

<script>
    import * as moment from 'moment'

    export default {
        name: 'Block',
        data() {
            return {
                block_num: 0,
                items: [
                    {
                        text: '首页',
                        to: {name: 'Home'}
                    }
                ],
                block: null
            }
        },
        created() {
            let self = this
            self.block_num = Number.parseInt(self.$route.params.block_num)
            self.items.push(
                {
                    text: `区块 # ${self.block_num}`
                }
            )
            if (self.block_num != 0) {
                self.GetInfo()
            }
        },
        watch: {
            // 如果路由有变化，会再次执行该方法
            '$route': function () {
                this.block_num = Number.parseInt(this.$route.params.block_num)
                this.GetInfo()
            }
        },
        methods: {
            GetMoment: function (date) {
                return moment(date).utcOffset(960).format('YYYY-MM-DD HH:mm:ss')
            },
            GetInfo() {
                let self = this
                self.$parent.getBlock(self.block_num).then(r => {
                    // console.log(r)
                    self.block = r
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
    .block-text {
        margin: 3px 0 3px 0;
    }

    .block-tr {
        margin-bottom: 10px;
        padding: 10px 15px 10px 15px;
        overflow: hidden;
    }

    .block-tr:last-child {
        margin-bottom: 0;
    }
</style>
