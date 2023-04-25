<!--
 * @Description:
 * @Version: 2.0
 * @Author: yichuanhao
 * @Date: 2023-04-23 11:49:25
 * @LastEditors: yichuanhao
 * @LastEditTime: 2023-04-25 12:03:09
-->
<template>
  <div class="treeCharts">
    <div id="chart" :style="{ width: '100vw', height: '700px' }"></div>
    <!-- 是否展示为圆 -->
    <div class="fixed" @click="drawer = true">
      <i class="el-icon-s-operation"></i>
    </div>
    <el-drawer :visible.sync="drawer" direction="rtl" custom-class="treeD" :modal="false">
      <div class="switch">
        <span>颜色名称：</span>
        <el-input placeholder="请输入" v-model="colorName" style="font-size: 13px" size="small" @keydown.enter.native="queryData">
          <i slot="suffix" class="el-icon-search" @click="queryData"></i>
        </el-input>
      </div>
      <div class="switch">
        <span>字体大小：</span>
        <el-input-number v-model="fontSize" :step="1" :min="8" :max="13" size="small" style="width: 192px"></el-input-number>
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
        <span>高亮颜色：</span>
        <el-color-picker v-model="eColor"></el-color-picker>
      </div>
      <div class="switch">
        <span>文字颜色：</span>
        <el-color-picker v-model="fontColor"></el-color-picker>
      </div>
    </el-drawer>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import Papa from 'papaparse';
export default {
  name: 'threeDPage', // 层级树
  data() {
    return {
      drawer: false,
      colorName: '',
      fontSize: 9,
      lineColor: '#fff',
      fontColor: '#fff',
      eColor: '#42cccc',
      currentDataIndexs: [], // 高亮节点index
      status: true,
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
            layout: 'radial', // 设置为 radial 布局方式
            symbol: 'emptyCircle',
            symbolSize: 7,
            initialTreeDepth: 4,
            animationDurationUpdate: 750,
            emphasis: {
              lineStyle: {
                color: '#42cccc',
              },
            },
            label: {
              fontSize: 9,
              color: '#fff',
            },
          },
        ],
      },
    };
  },
  methods: {
    queryData() {
      this.curstomDom.dispatchAction({
        type: 'downplay',
        dataIndex: this.currentDataIndexs,
      });
      if (!this.colorName) return;
      let data = this.findPath(this.colorName, this.firstLevelList);
      if (data && data.length > 0) {
        let arr = data.map((item) => item.index);
        this.currentDataIndexs = [0, ...arr];
        // 高亮点击已保存的相关节点的连线，防止上一步取消了已保存节点的高亮
        this.curstomDom.dispatchAction({
          type: 'highlight',
          dataIndex: this.currentDataIndexs,
        });
      }
    },
    findPath(targetValue, tree, path = []) {
      if (tree.name === targetValue) {
        return path.concat([tree]);
      }
      if (tree.children) {
        for (let i = 0; i < tree.children.length; i++) {
          const child = tree.children[i];
          const result = this.findPath(targetValue, child, path.concat([tree]));
          if (result) {
            return result;
          }
        }
      }
      return null;
    },
    getBarParams() {
      this.option.series[0].data = [this.firstLevelList];
      this.renderBar();
    },
    //渲染
    renderBar() {
      this.curstomDom.setOption(this.option); //设置配置项
      this.$nextTick(() => {
        this.initOtherEvent();
      });
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
      let index = 1;
      this.firstLevelList.forEach((item) => {
        index += 1;
        item.index = index;
        item.name = item.color;
        item.children = [];
        this.secondLevelList.forEach((val) => {
          val.name = val.color;
          if (val.parent == item.color) {
            index += 1;
            val.index = index;
            item.children.push(val);
          }
        });
      });
      this.firstLevelList = {
        index: 1,
        name: '中国传统色',
        children: this.firstLevelList,
      };
      this.getBarParams();
    },
    initOtherEvent() {
      let that = this;
      this.curstomDom.on('click', function (params) {
        // 先取消已高亮的节点连线
        that.curstomDom.dispatchAction({
          type: 'downplay',
          dataIndex: that.currentDataIndexs,
        });
        const treeAncestors = params.treeAncestors;
        const dataIndexs = treeAncestors.map((item) => item.dataIndex);
        // 针对非最尾节点点击收缩展开后已高亮线路失效
        if (
          dataIndexs.length > 0 &&
          that.currentDataIndexs.length > 0 &&
          dataIndexs[dataIndexs.length - 1] === that.currentDataIndexs[that.currentDataIndexs.length - 1]
        ) {
          that.currentDataIndexs = [];
          return;
        }
        if (params.data.children) {
          that.currentDataIndexs = [];
          return;
        }
        // 重新保存当前高亮的节点
        that.currentDataIndexs = dataIndexs;
        // 高亮相关节点连线
        that.curstomDom.dispatchAction({
          type: 'highlight',
          dataIndex: dataIndexs,
        });
      });
      // 节点鼠标移入事件
      this.curstomDom.on('mouseover', function (params) {
        // 取消当前节点的高点，顶替默认事件
        that.curstomDom.dispatchAction({
          type: 'downplay',
          dataIndex: params.dataIndex,
        });

        // 高亮点击已保存的相关节点的连线，防止上一步取消了已保存节点的高亮
        that.curstomDom.dispatchAction({
          type: 'highlight',
          dataIndex: that.currentDataIndexs,
        });
      });
    },
    //改变chart大小
    changeChartSize() {
      this.curstomDom.resize();
    },
    statusChange() {
      if (this.status) {
        this.option.series = [
          {
            type: 'tree',
            data: [this.firstLevelList],
            left: '2%',
            right: '2%',
            lineStyle: {
              color: this.lineColor,
            },
            layout: 'radial', // 设置为 radial 布局方式
            symbol: 'emptyCircle',
            symbolSize: 7,
            initialTreeDepth: 4,
            animationDurationUpdate: 750,
            emphasis: {
              lineStyle: {
                color: this.eColor,
              },
            },
            label: {
              fontSize: this.fontSize,
              color: this.fontColor,
            },
          },
        ];
      } else {
        this.option.series = [
          {
            type: 'tree',
            data: [this.firstLevelList],
            left: '2%',
            right: '2%',
            lineStyle: {
              color: this.lineColor,
            },
            layout: '',
            symbol: 'emptyCircle',
            orient: 'vertical',
            symbolSize: 7,
            initialTreeDepth: 4,
            animationDurationUpdate: 750,
            emphasis: {
              lineStyle: {
                color: this.eColor,
              },
            },
            label: {
              fontSize: this.fontSize,
              color: this.fontColor,
              position: 'left',
              verticalAlign: 'middle',
              align: 'right',
            },
            leaves: {
              label: {
                position: 'bottom',
                rotate: -90,
                verticalAlign: 'middle',
                align: 'left',
                fontSize: this.fontSize,
                color: this.fontColor,
              },
            },
          },
        ];
      }
      this.curstomDom.clear();
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
        if (!this.status) this.option.series[0].leaves.label.color = val ? val : '#fff';
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    eColor: function (val) {
      this.$nextTick(() => {
        this.option.series[0].emphasis.lineStyle.color = val ? val : '#42cccc';
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    fontSize: function (val) {
      this.$nextTick(() => {
        this.option.series[0].label.fontSize = val;
        if (!this.status) this.option.series[0].leaves.label.fontSize = val;
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
  },
};
</script>

<style lang="scss">
.treeCharts {
  .fixed {
    position: absolute;
    right: 10px;
    top: 10px;
    i {
      cursor: pointer;
      color: #fff;
    }
  }
}
.treeD {
  padding: 0 10px;
  width: 240px !important;
  .switch {
    .el-icon-search {
      line-height: 32px;
    }
    color: #fff;
    text-align: left;
    display: flex;
    align-items: center;
    margin-bottom: 10px;
    span {
      color: #000;
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
  .el-drawer__header {
    padding: 10px 6px 0;
  }
}
</style>
