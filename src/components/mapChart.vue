<template>
  <div>
    <div>
      <el-button size="mini" type="primary" @click="backMap()" class="btn">返回概览</el-button>
      <div id="container"></div>
    </div>
  </div>
</template>
<script>
import * as echarts from 'echarts';
import $ from "jquery";

require("echarts/lib/chart/map");
import {CITY_MAP, AREA_MAP} from './map/zhejiang-main-city-map'

export default {
  name: "mapChart",
  data() {
    return {
      myChart: '',
      provinceCode: 'zhejiang',
      provinceName: '浙江',
    };
  },
  mounted() {
    // 初始化加载
    this.mapChart();
    this.showProvince()
  },
  methods: {
    // 返回全国视图
    backMap() {
      this.showProvince();
    },
    showProvince() {
      let eName = this.provinceCode
      let param = this.provinceName
      $.getJSON(`/map/province/${eName}.json`, data => {
        this.$echarts.registerMap(param, data);
        this.initEcharts(param);
      })
    },
    initEcharts(map) {
      let option = {
        tooltip: {
          trigger: 'item',
          formatter: '{b}<br/>{c} (p / km2)'
        },
        series: [
          {
            name: "信息量",
            type: "map",
            mapType: map,
            selectedMode: 'single',
            showLegendSymbol: false,
            visibility: 'off',
            data: [
              {name: '杭州市', value: 20057.34},
              {name: '宁波市', value: 15477.48}
            ],
            label: {
              show: true,
              formatter: function (val) {
                var area_content = '{a|' + val.name + '}' + '-' + '{b|' + val.value + '}';
                return area_content.split("-").join("\n");
              }, rich: {
                a: {
                  color: 'black'
                },
                b: {
                  color: 'blue',
                  fontSize: 14,
                }
              }
            }
          }
        ]
      };

      this.myChart.setOption(option);
    },
    // 配置渲染map
    mapChart() {
      this.myChart = echarts.init(document.getElementById("container"));
      window.addEventListener("resize", () => {
        this.myChart.resize();
      });
      // 点击触发
      this.myChart.on("click", param => {
        if (param.name in CITY_MAP) {
          // 处理市模块
          let names = param.name;
          for (let key in CITY_MAP) {
            if (names === key) {
              this.showCitys(CITY_MAP[key], key);
              break;
            }
          }
        } else if (param.name in AREA_MAP) {
          // 处理区域模块
          let names = param.name;
          for (let key in AREA_MAP) {
            if (names === key) {
              this.showArea(AREA_MAP[key], key);
              break;
            }
          }
        }
      });
    },
    //展示对应市
    showCitys(cName, param) {
      // 显示县级地图
      $.getJSON(`/map/city/${cName}.json`, data => {
        this.$echarts.registerMap(param, data);
        this.initEcharts(param);
      })
    },
    //展示对应市
    showArea(aName, param) {
      // 显示县级地图
      $.getJSON(`/map/area/${aName}.json`, data => {
        this.$echarts.registerMap(param, data);
        this.initEcharts(param);
      })
    }
  }
};
</script>

<style scoped>
#container {
  width: 700px;
  height: 800px;
  margin: 0px auto 0;
}

.btn {
  position: absolute;
  right: 10%;
  z-index: 999;
}
</style>


