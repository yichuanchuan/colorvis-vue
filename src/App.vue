<!--
 * @Description:
 * @Version: 2.0
 * @Author: yichuanhao
 * @Date: 2023-04-23 10:50:51
 * @LastEditors: yichuanhao
 * @LastEditTime: 2023-04-26 14:26:12
-->
<template>
  <div id="app">
    <div class="container">
      <p class="container-title">中国传统色</p>
      <el-tabs v-model="activeName">
        <el-tab-pane label="全部颜色" name="first">
          <threeDPage :backGroudColor="backGroudColor" />
        </el-tab-pane>
        <el-tab-pane label="色名关系" name="second">
          <treeChart />
        </el-tab-pane>
        <el-tab-pane label="色名列表" name="third">
          <treeTable />
        </el-tab-pane>
      </el-tabs>
      <div class="backGroud">
        <el-color-picker v-model="backGroudColor"></el-color-picker>
      </div>
    </div>
  </div>
</template>

<script>
import threeDPage from './components/threeDPage.vue';
import treeChart from './components/treeChart.vue';
import treeTable from './components/treeTable.vue';
export default {
  name: 'App',
  components: { threeDPage, treeChart, treeTable },
  data() {
    return {
      activeName: 'first',
      backGroudColor: '#010101',
    };
  },
  watch: {
    backGroudColor: function (val) {
      this.$nextTick(() => {
        document.querySelector('.el-tabs__content').style.background = val ? val : '#010101';
      });
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
* {
  margin: 0;
  padding: 0;
}
.container {
  position: relative;
  .container-title {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 1000;
    width: 140px;
    text-align: center;
    height: 48px;
    line-height: 48px;
    font-size: 16px;
    background: rgb(49, 49, 49);
    color: #fff;
  }
}
.backGroud {
  position: fixed;
  right: 10px;
  top: 10px;
  .el-color-picker__trigger {
    width: 30px;
    height: 30px;
  }
}
.el-tabs__header {
  margin: 0 !important;
  padding-left: 140px !important;
  background: rgb(49, 49, 49);
  .el-tabs__item {
    height: 50px;
    line-height: 50px;
    color: rgb(194, 194, 194);
    &:hover {
      color: #fff;
    }
  }
  .is-active {
    color: #fff !important;
  }
  .el-tabs__active-bar {
    background-color: #fff;
    bottom: 2px;
  }
}
.el-tabs__nav-wrap::after {
  height: 0 !important;
}
.el-tabs__content {
  height: calc(100vh - 50px) !important;
  overflow: auto !important;
  background: rgb(1, 1, 1);
  &::-webkit-scrollbar {
    width: 0px;
  }
  &::-webkit-scrollbar-thumb {
    border-radius: 5px;
    background: rgb(49, 49, 49);
  }
  &::-webkit-scrollbar-track {
    border-radius: 0;
    background: rgb(1, 1, 1);
  }
}
</style>
