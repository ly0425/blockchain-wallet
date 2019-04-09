<template>
    <div class="hello">
        <div style="padding: 20px">
            <canvas id="chart" width="2000" height="1000"></canvas>
        </div>
        <div class="search">
            <div>
                <Select v-model="base" style="width:200px">
                    <Option value="eth" key="eth">ETH</Option>
                    <Option value="btc" key="btc">BTC</Option>
                </Select>
                <Input class="period" placeholder="查询间隔" value="5" v-model="period"/>
            </div>
        </div>
        <div class="list">
            <Table :columns="columns" :data="data"></Table>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    import moment from 'moment';
    export default {
        name: 'HelloWorld',
        props: {
            msg: String
        },
        data() {
            return {
                tether: [],
                ticker: [],
                columns: [
                    {
                        title: '货币',
                        key: 'currency'
                    },
                    {
                        title: '市场价(15分钟内)',
                        key: '15m'
                    },
                    {
                        title: '最终',
                        key: 'last'
                    },
                    {
                        title: '买入',
                        key: 'buy'
                    },
                    {
                        title: '卖出',
                        key: 'sell'
                    },
                    {
                        title: '货币符号',
                        key: 'symbol'
                    }
                ],
                data: [],
                base: 'eth',
                period: 5,
                timer: null,
                chart: 'chart',
                context: {},
                lineColor: 'red',
            }
        },
        methods: {
            getTicker() {
                const vue = this;
                axios.get(`https://api.blockchain.info/ticker?currency=CNY&base=${this.base}&cors=true`)
                    .then(function (response) {
                        console.log(response);
                        const data = response.data;
                        Object.keys(data).forEach(key => {
                            const item = data[key];
                            item.currency = key;
                            vue.data.push(item);
                        });
                    })
                    .catch(function (error) {
                        console.log(error);
                    });
            },
            init() {
                //clearInterval(this.timer)
                const vue = this;
                setInterval(function () {
                    vue.getTicker(vue.base)
                }, vue.period * 1000);
            },
            initDraw() {
                // 初始化画布
                const canvas = document.getElementById("chart");
                this.context = canvas.getContext('2d')
                //this.draw(this.context, 0, 0, 50, 50, "#FF0000");

            },
            draw(context, x1, y1, x2, y2, color) {
                context.fillStyle = color;
                context.fillRect(x1, y1, x2, y2);
            }
        },
        watch: {
            base() {
                this.init();
            },
            period() {
                this.init();
            },
            tether(val){
                for (var i = 0; i < 100; i++) {
                    this.draw(this.context, i * 10, val[i].high * 10000 - 40, 10, 10, "#FF0000")
                }
            }
        },
        mounted() {
            const vue = this;
            axios.get('https://api.coincap.io/v2/candles?exchange=poloniex&interval=h8&baseId=ethereum&quoteId=bitcoin')
                    .then(function (response) {
                      console.log(response);
                      vue.tether = response.data.data;
                    })
                    .catch(function (error) {
                      console.log(error);
                    });
            //this.getTicker();
            //this.init();
            this.initDraw();
        },
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }

    .search {

    }

    .search .period {
        width: 100px;
        margin-left: 20px;
    }

    .list {
        padding: 10px 200px;
        margin-top: 20px;
    }
    #chart{
        width:1000px;height:500px;
    }
</style>
