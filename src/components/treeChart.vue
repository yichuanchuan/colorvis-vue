<!--
 * @Description:
 * @Version: 2.0
 * @Author: yichuanhao
 * @Date: 2023-04-23 11:49:25
 * @LastEditors: yichuanhao
 * @LastEditTime: 2023-04-23 17:28:41
-->
<template>
  <div class="treeCharts">
    <div id="chart" :style="{ width: '100vw', height: '500px' }"></div>
    <!-- 是否展示为圆 -->
    <div class="box">
      <div class="switch">
        <span>颜色名称：</span>
        <el-input placeholder="请输入" v-model="colorName" style="font-size: 13px" size="small" @keydown.enter.native="queryData">
          <i slot="suffix" class="el-icon-search" @click="queryData"></i>
        </el-input>
      </div>
      <div class="switch">
        <span>是否展示为圆：</span>
        <el-switch
          v-model="status"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-text="status ? '是' : '否'"
          @change="statusChange"
        ></el-switch>
      </div>
      <div class="switch">
        <span>线条颜色：</span>
        <el-color-picker v-model="lineColor"></el-color-picker>
      </div>
      <div class="switch">
        <span>文字颜色：</span>
        <el-color-picker v-model="fontColor"></el-color-picker>
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import Papa from 'papaparse';
export default {
  name: 'threeDPage', // 层级树
  data() {
    return {
      colorName: '',
      lineColor: '#fff',
      fontColor: '#fff',
      status: false,
      firstLevelList: [], // 第一层级列表
      secondLevelList: [], // 第二层级列表
      thirdLevelList: [], // 第三层级列表
      listLength: 0,
      curstomDom: '', //柱状图dom
      option: {
        tooltip: {
          trigger: 'item',
          triggerOn: 'mousemove',
        },
        series: [
          {
            type: 'tree',
            data: [],
            left: '2%',
            right: '2%',
            lineStyle: {
              color: '#fff',
            },
            top: '8%',
            bottom: '20%',
            symbol: 'emptyCircle',
            orient: 'vertical',
            expandAndCollapse: true,
            label: {
              position: 'top',
              rotate: -90,
              verticalAlign: 'middle',
              align: 'right',
              fontSize: 9,
              color: '#fff',
            },
            leaves: {
              label: {
                position: 'bottom',
                rotate: -90,
                verticalAlign: 'middle',
                align: 'left',
              },
            },
            animationDurationUpdate: 750,
          },
        ],
      },
    };
  },
  methods: {
    queryData() {
      if (!this.colorName) return;
      console.log(this.colorName);
    },
    getBarParams() {
      this.option.series[0].data = this.firstLevelList;
      this.renderBar();
    },
    //渲染
    renderBar() {
      this.curstomDom.setOption(this.option); //设置配置项
    },
    parseCsvData(url, key) {
      let that = this;
      Papa.parse(url, {
        download: true,
        header: true,
        complete: (results) => {
          this[key] = results.data;
          that.listLength++;
        },
      });
    },
    levelListHandel() {
      this.firstLevelList.forEach((item) => {
        item.name = item.color;
        item.children = [];
        this.secondLevelList.forEach((val) => {
          val.name = val.color;
          if (val.parent == item.color) {
            item.children.push(val);
          }
        });
      });
      this.getBarParams();
    },
    //改变chart大小
    changeChartSize() {
      this.curstomDom.resize();
    },
    statusChange() {
      if (this.status) {
        this.option.series[0].layout = 'radial';
      } else {
        this.option.series[0].layout = '';
      }
      this.curstomDom.setOption(this.option); //设置配置项
    },
  },
  created() {
    this.index = 0;
    this.parseCsvData('/assets/color/firstLevel.csv', 'firstLevelList');
    this.parseCsvData('/assets/color/secondLevel.csv', 'secondLevelList');
    this.parseCsvData('/assets/color/thirdLevel.csv', 'thirdLevelList');
  },
  mounted() {
    // 柱状图实例化
    this.curstomDom = echarts.init(document.querySelector('#chart'));
    // 监听resize
    window.addEventListener('resize', this.changeChartSize);
  },
  watch: {
    listLength: function (val) {
      if (val === 3) {
        this.levelListHandel();
      }
    },
    lineColor: function (val) {
      this.$nextTick(() => {
        this.option.series[0].lineStyle.color = val ? val : '#fff';
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    fontColor: function (val) {
      this.$nextTick(() => {
        this.option.series[0].label.color = val ? val : '#fff';
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
  },
};
</script>

<style lang="scss">
.treeCharts {
  .el-icon-search {
    line-height: 32px;
  }
  .box {
    position: absolute;
    top: 15px;
    left: 20px;
  }
  .switch {
    color: #fff;
    text-align: left;
    display: flex;
    align-items: center;
    margin-bottom: 10px;
    span {
      white-space: nowrap;
      font-size: 14px;
    }
  }
  .el-color-picker {
    height: 25px;
  }
  .el-color-picker__trigger {
    width: 25px;
    height: 25px;
  }
}
</style>
