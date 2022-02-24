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
      markerList: [
        {value:10,name:'杭州萧山瓜沥沈天海蛇类养殖场',latitude: 30.206742,longitude: 120.423629},
        {value:20,name:'杭州萧山鑫怡农业开发有限公司',latitude: 30.095659,longitude: 120.17343},
        {value:30,name:'杭州长乔旅游投资集团股份有限公司',latitude: 30.143301,longitude: 120.221168},
        {value:40,name:'杭州萧山珍禽养殖有限公司',latitude: 30.200539,longitude: 120.386077},
        {value:50,name:'杭州萧山湘湖鸟语林科普有限公司',latitude: 30.143301,longitude: 120.221168}
      ],
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
      console.log("map",map)
      let option = {
        tooltip: {
          trigger: 'item',
          formatter: '{b}<br/>{c} (p / km2)'
        },
        geo: {
          map: map,
          roam: true,//是否开启缩放和平移
          zoom: 1.2,//视角缩放比例
          label: {
            normal: {
              show: true,//是否显示省份名称
              fontSize: '10',//字体大小
              color: '#000'//字体颜色
            },
            emphasis: { //动态展示的样式
              fontSize: '10',//字体大小
              color: '#fff'//字体颜色
            },
          },
          itemStyle: {
            normal: {
              borderColor: 'rgba(0, 0, 0, 0.2)',
              areaColor: '#BBFFFF',//静态时各省份区域颜色
            },
            emphasis: {
              areaColor: '#3c92cf',//鼠标选择区域颜色
              shadowOffsetX: 0,
              shadowOffsetY: 0,
              shadowBlur: 20,
              borderWidth: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          }
        },
        series: [
          {
            type: 'effectScatter',
            coordinateSystem: 'geo',
            animation: false,//坐标点是否显示动画
            // symbol:'pin',
            data: this.markerList.map(function (itemOpt) {
              return {
                name: itemOpt.name,
                value: [
                  itemOpt.longitude,
                  itemOpt.latitude,
                  itemOpt.value //数量
                ],
                label: {
                  emphasis: {
                    position: 'right',
                    show: false
                  }
                },
                itemStyle: {
                  normal: {
                    color: '#0394d9'
                  }
                }
              };
            }),
            // name: "信息量",
            // type: "map",
            // mapType: map,
            // selectedMode: 'single',
            // showLegendSymbol: false,
            // visibility: 'off',
            // data: [
            //   {name: '杭州市', value: 20057.34},
            //   {name: '宁波市', value: 15477.48}
            // ],
            // symbolSize: function (val) {
            //   console.log(val)
            //   return 5;//描点的大小
            // },
            // label: {
            //   show: true,
            //   formatter: function (val) {
            //     var area_content = '{a|' + val.name + '}' + '-' + '{b|' + val.value + '}';
            //     return area_content.split("-").join("\n");
            //   }, rich: {
            //     a: {
            //       color: 'black'
            //     },
            //     b: {
            //       color: 'blue',
            //       fontSize: 14,
            //     }
            //   }
            // }
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
        console.log("param",param)
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


