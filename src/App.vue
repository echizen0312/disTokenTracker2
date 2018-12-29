<template>
    <div>
        <b-navbar toggleable="md" type="dark" variant="secondary">
            <b-navbar-brand style="letter-spacing: 2px; font-weight: 600;">{{title}}</b-navbar-brand>
            <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
            <b-collapse is-nav id="nav_collapse">
                <b-navbar-nav>
                    <b-nav-item>&nbsp;</b-nav-item>
                </b-navbar-nav>
                <b-navbar-nav class="ml-auto">
                    <b-nav-form>
                        <b-form-input type="text" placeholder="区块 / 交易 / 账户"
                                      v-model="keyword"
                                      style="min-width: 280px; max-width: 360px; margin-right: 10px;"/>
                        <b-button type="button" style="margin-right: 10px;" @click="GoSearch">查询</b-button>
                    </b-nav-form>
                    <b-nav-item-dropdown :text="netName" right>
                        <b-dropdown-item>{{netName}}</b-dropdown-item>
                    </b-nav-item-dropdown>
                </b-navbar-nav>
            </b-collapse>
        </b-navbar>
        <div style="max-width: 1000px; margin: 8px auto; padding: 8px;">
            <router-view/>
        </div>
    </div>
</template>

<script>
    let EosApi = require('eosjs-api') // 引入只读EOS

    export default {
        name: 'App',
        data() {
            return {
                title: 'disTokenTracker',
                // netId: '001',
                // netName: 'testNet',
                // chainId: 'cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f',
                netId: '999',
                netName: 'MainNet',
                chainId: 'aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906',
                httpEndpoint: 'https://eos.greymass.com',
                keyword: '',
                eosApi: null
            }
        },
        created() {
            let self = this
            self.eosApi = EosApi({
                chainId: self.chainId,
                httpEndpoint: self.httpEndpoint
            })
        },
        watch: {},
        methods: {
            GoSearch() {
                let self = this
                let kw = self.keyword
                if (!isNaN(kw)) {
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
            //======================== api =========================
            getBlock(bid) {
                let self = this
                return self.eosApi.getBlock(bid)
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
</style>
