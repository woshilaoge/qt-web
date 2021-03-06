<template>
    <div class="echarts" :style="{'height':height+'px'}"></div>
</template>
<style>
.echarts {
    width: 100%;
    height: 400px;
}
</style>
<script>
// import echarts from 'echarts/lib/echarts'
import lodash from 'lodash'
let debounce = lodash.debounce;
// enumerating ECharts events for now
const ACTION_EVENTS = [
    'legendselectchanged',
    'legendselected',
    'legendunselected',
    'datazoom',
    'datarangeselected',
    'timelinechanged',
    'timelineplaychanged',
    'restore',
    'dataviewchanged',
    'magictypechanged',
    'geoselectchanged',
    'geoselected',
    'geounselected',
    'pieselectchanged',
    'pieselected',
    'pieunselected',
    'mapselectchanged',
    'mapselected',
    'mapunselected',
    'axisareaselected',
    'brush',
    'brushselected'
]
const MOUSE_EVENTS = [
        'click',
        'dblclick',
        'mouseover',
        'mouseout',
        'mousedown',
        'mouseup',
        'globalout'
    ]
    //可针对 主题进行配置个性化loading
const LOADING = {
    text: 'loading',
    color: '#c23531',
    textColor: '#000',
    maskColor: 'rgba(255, 255, 255, 0.8)',
    zlevel: 0
}

export default {
    props: {
        options: {
            type: Object,
            default: null
        },
        theme: String,
        initOptions: {
            type: Object,
            default: null
        },
        group: String,
        height: {
            type: [Number, String],
            default: '400'
        },
        autoResize: Boolean
    },
    data() {
        return {
            chart: null
        }
    },
    computed: {
        // Only recalculated when accessed from JavaScript.
        // Won't update DOM on value change because getters
        // don't depend on reactive values
        //        width: {
        //            cache: false,
        //            getter() {
        //                return this.chart.getWidth()
        //            }
        //        },
        //        height: {
        //            cache: false,
        //            getter() {
        //                return this.chart.getHeight()
        //                return this.chart.getHeight()
        //                return this.chart.getHeight()
        //                return this.chart.getHeight()
        //                return this.chart.getHeight()
        //                return this.chart.getHeight()
        //            }
        //        },
        //        isDisposed: {
        //            cache: false,
        //            getter() {
        //                return this.chart.isDisposed()
        //            }
        //        }
    },
    watch: {
        options: {
            handler(options) {
                console.log(options, 'options=========999');
                if (options) {
                    this.chart.hideLoading();
                    this.chart.setOption(options, true);
                }
            },
            deep: true
        },
        group: {
            handler(group) {
                this.chart.group = group
            }
        }
    },
    methods: {
        // provide a explicit merge option method
        mergeOptions(options) {
            this.chart.hideLoading();
            this.chart.setOption(options)
        },
        // just delegates ECharts methods to Vue component
        resize(options) {
            this.chart.resize(options)
        },
        dispatchAction(payload) {
            this.chart.dispatchAction(payload)
        },
        convertToPixel(...args) {
            return this.chart.convertToPixel(...args)
        },
        convertFromPixel(...args) {
            return this.chart.convertFromPixel(...args)
        },
        containPixel(...args) {
            return this.chart.containPixel(...args)
        },
        showLoading(...args) {
            this.chart.showLoading(...args)
        },
        hideLoading() {
            this.chart.hideLoading()
        },
        getDataURL(options) {
            return this.chart.getDataURL(options)
        },
        getConnectedDataURL(options) {
            return this.chart.getConnectedDataURL(options)
        },
        clear() {
            this.chart.clear()
        },
        dispose() {
            this.chart.dispose()
        }
    },
    mounted() {
        let chart = echarts.init(this.$el, this.theme, this.initOptions)
            // use assign statements to tigger "options" and "group" setters

        //默认不初始化空表格，默认出loading
        chart.showLoading(LOADING);
        // chart.setOption(this.options)
        // expose ECharts events as custom events
        ACTION_EVENTS.forEach(event => {
                chart.on(event, params => {
                    this.$emit(event, params)
                })
            })
            // mouse events of ECharts should be renamed to prevent
            // name collision with DOM events
        MOUSE_EVENTS.forEach(event => {
            chart.on(event, params => {
                this.$emit('chart' + event, params)
            })
        })
        if (this.autoResize) {
            this.__resizeHanlder = debounce(() => {
                chart.resize()
            }, 100, {
                leading: true
            })
            window.addEventListener('resize', this.__resizeHanlder)
        }
        this.chart = chart;
        if(this.options){
            this.mergeOptions(this.options);
        }
    },
    connect(group) {
        if (typeof group !== 'string') {
            group = group.map(chart => chart.chart)
        }
        echarts.connect(group)
    },
    disconnect(group) {
        echarts.disconnect(group)
    },
    registerMap(...args) {
        echarts.registerMap(...args)
    },
    registerTheme(...args) {
        echarts.registerTheme(...args)
    }
}
</script>
