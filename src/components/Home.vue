<template>
    <div>
        <b-card v-if="info != null"
                :title="`区块链信息`"
                tag="article"
                style=""
                class="mb-2">
            <p class="card-text">
                <b-row>
                    <b-col xs="12" sm="12" md="6" lg="3" class="home-text">
                        <div style="margin-bottom: 4px;">链名称</div>
                        <div style="font-size: 20px;">{{$parent.netConfig.netName}}</div>
                    </b-col>
                    <b-col xs="12" sm="12" md="6" lg="3" class="home-text">
                        <div style="margin-bottom: 4px;">最新区块</div>
                        <div style="font-size: 20px;">
                            <span style="cursor: pointer; color: #007bff;" @click="GoBlock(info.head_block_num)">{{info.head_block_num}}</span>
                        </div>
                    </b-col>
                    <b-col xs="12" sm="12" md="6" lg="3" class="home-text">
                        <div style="margin-bottom: 4px;">生产者</div>
                        <div style="font-size: 20px;">{{info.head_block_producer}}</div>
                    </b-col>
                    <b-col xs="12" sm="12" md="6" lg="3" class="home-text">
                        <div style="margin-bottom: 4px;">生产时间</div>
                        <div style="font-size: 20px;">{{GetMoment(info.head_block_time, 960)}}</div>
                    </b-col>
                </b-row>
            </p>
        </b-card>
        <b-card :title="`最新交易`"
                tag="article"
                style="margin-top: 16px; font-size: 14px;"
                class="mb-2">
            <p class="card-text" style="padding-top: 12px;">
                <b-form-checkbox v-if="$parent.netConfig.netId != '999'" v-model="isAuto"
                                 style="position: absolute; top: 24px; right: 6px; font-size: 16px;">
                    自动刷新
                </b-form-checkbox>
                <b-card v-if="$parent.netConfig.netId == '999'" no-body class="home-tr">
                    <b-row>
                        <b-col xs="12" sm="12">
                            MainNet暂不提供最新交易列表
                        </b-col>
                    </b-row>
                </b-card>
                <template v-else>
                    <b-card no-body class="home-tr d-none d-md-block d-lg-block">
                        <b-row>
                            <b-col xs="12" sm="6" md="7">
                                交易
                            </b-col>
                            <b-col xs="12" sm="2" md="2" style="text-align: center;">
                                Actions
                            </b-col>
                            <b-col xs="12" sm="4" md="3" style="text-align: center;">
                                交易时间
                            </b-col>
                        </b-row>
                    </b-card>
                    <b-card no-body v-for="(tr, index) in newTrx" :key="'tr_' + index"
                            class="home-tr">
                        <b-row>
                            <b-col xs="12" sm="6" md="7" class="sak-text">
                            <span style="cursor: pointer; color: #007bff;" @click="GoTransaction(tr.trx_id)">
                                {{tr.trx_id}}
                            </span>
                            </b-col>
                            <b-col xs="12" sm="2" md="2" style="text-align: center;">
                                {{tr.actions}}
                            </b-col>
                            <b-col xs="12" sm="4" md="3" style="text-align: center;">
                                {{GetMoment(tr.createdAt, 480)}}
                            </b-col>
                        </b-row>
                    </b-card>
                </template>
            </p>
        </b-card>
    </div>
</template>

<script>
    import * as moment from 'moment'

    export default {
        name: 'Home',
        data() {
            return {
                info: null,
                newTrx: [],
                isAuto: false
            }
        },
        created() {
            let self = this
            self.GetInfo(true)
            setInterval(() => {
                if (self.isAuto) {
                    self.GetInfo(false)
                }
            }, 3000)
        },
        watch: {
            // 如果路由有变化，会再次执行该方法
            // '$route': function () {
            //     this.GetInfo()
            // }
        },
        methods: {
            GetMoment: function (date, offset) {
                return moment(date).utcOffset(offset).format('YYYY-MM-DD HH:mm:ss')
            },
            GetInfo(mask) {
                let self = this
                if (mask) {
                    self.$parent.StartLoading()
                }
                let p1 = self.$parent.getInfo().then(r => {
                    // console.log(r)
                    self.info = r
                }).catch(e => {
                    console.log(e)
                })
                let p2 = self.$parent.getNewTrx().then(r => {
                    // console.log(r)
                    let data = r.body
                    if (data.success) {
                        self.newTrx = data.result
                    } else {
                        self.newTrx = []
                    }
                }).catch(e => {
                    console.log(e)
                })
                Promise.all([p1, p2]).then(() => {
                    self.$parent.StopLoading()
                }).catch(() => {
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
        }
    }
</script>

<style scoped>
    .home-text {
        margin: 3px 0 3px 0;
        font-size: 15px;
    }

    .home-tr {
        margin-bottom: 10px;
        padding: 10px 15px 10px 15px;
        overflow: hidden;
    }

    .home-tr:last-child {
        margin-bottom: 0;
    }
</style>
