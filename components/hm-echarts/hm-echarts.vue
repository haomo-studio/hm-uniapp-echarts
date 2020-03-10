<template>
  <div :id="dataId" class="echarts"></div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: "HmEcharts",
  components: {},
  props: {
    dataId: {
      type: String,
      default: "echarts",
    },
    dataUrl: {
      type: String,
      default: null,
    },
    apiType: {
      type: String,
      default: null, // Loopback/Java
    },
    optionUrl: {
      type: String,
      default: null, //echarts的基本配置
    },
    options: {
      type: [Object, Function],
      default: {
        backgroundColor: "#FFFFFF",
        title: {
          text: "Customized Pie",
          left: "center",
          top: 20,
          textStyle: {
            color: "#ccc",
          },
        },
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b} : {c} ({d}%)",
        },
        visualMap: {
          show: false,
          min: 80,
          max: 600,
          inRange: {
            colorLightness: [0, 1],
          },
        },
        series: [
          {
            name: "访问来源",
            type: "pie",
            radius: "55%",
            center: ["50%", "50%"],
            data: [
              {
                value: 235,
                name: "视频广告",
              },
              {
                value: 274,
                name: "联盟广告",
              },
              {
                value: 310,
                name: "邮件营销",
              },
              {
                value: 335,
                name: "直接访问",
              },
              {
                value: 400,
                name: "搜索引擎",
              },
            ],
            roseType: "radius",
            label: {
              normal: {
                textStyle: {
                  color: "rgba(0, 0, 0, 0.3)",
                },
              },
            },
            labelLine: {
              normal: {
                lineStyle: {
                  color: "rgba(0, 0, 0, 0.3)",
                },
                smooth: 0.2,
                length: 10,
                length2: 20,
              },
            },
            itemStyle: {
              normal: {
                color: "#c23531",
                shadowBlur: 0,
                shadowColor: "rgba(0, 0, 0, 0.5)",
              },
            },
            animationType: "scale",
            animationEasing: "elasticOut",
          },
        ],
      }
    },
    registerMapType: {
      type: String,
      default: null, //注册的地图类型
    },
    registerMap: {
      type: String,
      default: null, //注册的地图
    },
    eventListeners: {
      type: Array,
      default: null, // 事件监听器
    },
  },
  data() {
    return {
      hmChart: null,
      option: null,
      params: {},
    };
  },
  created() {
    const self = this;
	  console.log(`hm-echarts: `, echarts);
  },
  mounted() {
    let self = this;
    console.log(`hm-echarts ${self.dataId} onLoad`);
    //设置新的option
    if (typeof self.options == 'function') {
      self.option = self.options();
    } else {
      self.option = self.options;
    }
    self.getOption().then(function(option) {
	  console.log('hm-echarts option: ', option);
	  self.hmChart = echarts.init(
	    document.getElementById(self.dataId)
	  );
    if (!self.registerMapType || !self.registerMap) {
      self.setChartOption(option);
      self.registerEventFunc(self.hmChart);
    } else {
      // 注册地图
      self.registerMapFunc(option);
    }
  });
  },
  methods: {
    getOption() {
      let self = this;
      return new Promise(function(resolve, reject) {
        if (!self.optionUrl) {
          resolve(self.option);
        } else {
          uni.request({
            url: self.optionUrl,
            success: resp => {
              self.option = resp.data;
              console.log("options",self.option)
              // resolve(options);
              if (!self.params.dataUrl) {
                // 未修改数据来源
                self.setOptionData(self.option, self.option.series[0].data);
              } else {
                // 修改了数据来源
                self.$axios.get(self.params.dataUrl).then(resp => {
                  self.setOptionData(self.option, resp);
                });
              }
              resolve(self.option);
            },
            fail: err => {
              console.error(err);
              reject(err);
            },
          });
        }
      });
    },

    /**
     * 渲染图表
     */
    setChartOption(option) {
	  console.log("渲染echarts",option)
	  
      this.hmChart.setOption(option);
    },

    /**
     * 注册chart的事件函数回调
     */
    registerEventFunc(chart) {
      let self = this;
      console.log("registerEventFunc: ", self.attributes);

      if (!self.eventListeners) {
        return;
      }

      self.eventListeners.forEach(listener => {
        console.log(`注册事件：`, listener);
        chart.on(listener.eventType, function(params) {
          eval(listener.func);
        });
      });
    },

    /**
     * 注册地图
     */
    registerMapFunc(option) {
      let self = this
      uni.request({
        url: self.registerMap,
        success: resp => {
          console.log(
            "registerMap: ",
            self.registerMapType,
            self.registerMap,
            resp.data
          );
          echarts.registerMap(self.registerMapType, resp.data);
          self.setChartOption(option);
          self.registerEventFunc(self.hmChart);
        },
        fail: err => {
          console.error(err);
        },
      });
    },

    /**
     * 从数据接口获取数据，并根据dataHook/optionHook修改数据
     */
    setOptionData(option, resp) {
      let self = this;
      // 不同apiType获取数据的方式是不同的
      let data = resp;
      if (self.apiType == "Loopback") {
        data = resp.data;
      }
      // 执行dataHook
      self.$emit("data-hook", data);
      // 替换数据
      self.$jsonpatch.apply(option, [
        {
          op: "replace",
          path: "/series/0/data",
          value: data,
        },
      ]);
      console.log("option: ", option);
    }
  },
};
</script>
<style>
@charset "UTF-8";

.echarts {
  width: 720rpx;
  height: 100%;
	/* #ifdef H5 */
  min-width: 335px;
  min-height: 250px;
	/* #endif */
  overflow: hidden;
  margin: 25rpx 15rpx 25rpx 15rpx;
  padding: 0px;
  box-shadow: 0px 10rpx 10rpx rgba(209, 213, 223, 0.5);
  border-radius: 5rpx;
}
</style>