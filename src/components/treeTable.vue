<!--
 * @Description:
 * @Version: 2.0
 * @Author: yichuanhao
 * @Date: 2023-04-26 14:25:14
 * @LastEditors: yichuanhao 1274816963@qq.com
 * @LastEditTime: 2023-04-26 19:12:48
-->
<template>
  <div class="tree_container">
    <div class="content">
      <div class="header-btn">
        <div class="button" @click="checkAll">全选</div>
        <div class="button" @click="removeAll">清除</div>
      </div>
      <!-- 一级五色调 -->
      <div class="first-color">
        <div class="title">一级五色调</div>
        <div class="color-box">
          <div
            class="color-item"
            v-for="(item, index) in firstLevelList"
            :key="index"
            :class="[item.isChecked ? 'isChecked' : '']"
            @click="clickFirst(item)"
            :style="{ background: item.isChecked ? item.hex : '#fff' }"
          >
            <div class="isDisabled" v-show="!item.isActive"></div>
            <div :style="{ border: `1px solid ${item.hex}`, background: item.isChecked ? '#fff' : item.hex }" class="icon">
              <i v-show="item.isChecked" class="el-icon-check" :style="{ color: item.hex, fontWeight: 900 }"></i>
            </div>
            <span>{{ item.color }}</span>
          </div>
        </div>
      </div>
      <!-- 二级五色调 -->
      <div class="second-color">
        <div class="title">
          <span>正色</span>
          <span>间色</span>
        </div>
        <div class="color-box">
          <div
            class="color-item"
            v-for="(item, index) in secondLevelList"
            :key="index"
            :class="[item.isChecked ? 'isChecked' : '']"
            @click="clickSecond(item)"
            :style="{ background: item.isChecked ? item.hex : '#fff' }"
          >
            <div class="isDisabled" v-show="!item.isActive"></div>
            <div :style="{ border: `1px solid ${item.hex}`, background: item.isChecked ? '#fff' : '' }" class="icon">
              <i v-show="item.isChecked" class="el-icon-check" :style="{ color: item.hex, fontWeight: 900 }"></i>
            </div>
            <span>{{ item.color }}</span>
          </div>
        </div>
      </div>
      <!-- 三级 -->
      <div class="third-color">
        <div class="title">三级颜色</div>
        <div class="third-content">
          <div class="color-box" v-for="(item, index) in thirdLevelList" :key="index">
            <div class="color-title">
              <div class="icon" :style="{ background: item.color }"></div>
              <span style="margin-left: 10px">{{ item.name }}</span>
            </div>
            <div class="color-content">
              <div class="content-item" v-for="(val, i) in item.children" :key="i">
                <div class="color-item" :style="{ background: val.hex }">
                  <span style="color: #fff; display: none; font-size: 12px" class="hex">{{
                    `Lab(${Number(val.l).toFixed(0)}, ${Number(val.a).toFixed(0)}, ${Number(val.b).toFixed(0)})`
                  }}</span>
                </div>
                <div class="color-name">{{ val.color }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Papa from 'papaparse';
import { colord, extend } from 'colord';
import labPlugin from 'colord/plugins/lab';
extend([labPlugin]);
export default {
  name: 'treeTable', // 色名列表
  data() {
    return {
      firstLevelList: [],
      secondLevelList: [],
      secondOhterLevel: [],
      secondActiveList: [],
      thirdLevelList: [],
      thirdOtherLevel: [],
    };
  },
  methods: {
    checkAll() {
      this.firstLevelList.forEach((item) => {
        item.isChecked = true;
      });
      this.secondLevelList.forEach((item) => {
        item.isChecked = true;
        item.isActive = true;
      });
      this.searchThidLevel();
    },
    removeAll() {
      this.firstLevelList.forEach((item) => {
        item.isChecked = false;
      });
      this.secondLevelList.forEach((item) => {
        item.isChecked = false;
        item.isActive = false;
      });
      this.thirdLevelList = [];
    },
    clickFirst(val) {
      val.isChecked = !val.isChecked;
      this.secondActiveList = [];
      this.firstLevelList.forEach((o) => {
        if (o.isChecked) {
          let arr = this.secondOhterLevel.filter((item) => {
            return item.parent === o.color;
          });
          let arrMap = arr.map((item) => {
            return item.color;
          });
          let newArr = this.secondActiveList.concat(arrMap);
          this.secondActiveList = [...new Set(newArr)];
        }
      });
      this.secondLevelList.forEach((item) => {
        if (this.secondActiveList.indexOf(item.color) !== -1) {
          item.isActive = true;
        } else {
          item.isActive = false;
          item.isChecked = false;
        }
      });
      this.searchThidLevel();
    },
    clickSecond(item) {
      if (!item.isActive) return;
      item.isChecked = !item.isChecked;
      this.searchThidLevel();
    },
    searchThidLevel() {
      let arr = this.secondLevelList.filter((item) => {
        return item.isChecked;
      });
      this.thirdLevelList = [];
      arr.forEach((item) => {
        let filterArr = this.thirdOtherLevel.filter((val) => {
          val.hex = colord({ l: val.l, a: val.a, b: val.b }).toHex();
          return val.parent === item.color;
        });
        this.thirdLevelList.push({
          name: item.color,
          color: colord({ l: item.l, a: item.a, b: item.b }).toHex(),
          children: filterArr,
        });
      });
    },
    parseCsvData(url, key) {
      Papa.parse(url, {
        download: true,
        header: true,
        complete: (results) => {
          if (key == 'secondOhterLevel' || key == 'thirdOtherLevel') {
            this[key] = results.data;
          } else {
            this[key] = this.formatData(results.data, key == 'firstLevelList');
          }
        },
      });
    },
    // 处理数据给下拉框
    formatData(data, flag = false) {
      let op = [];
      data.forEach((d) => {
        let name = d.color.replace(/\r\n/, '');
        let color = colord({ l: d.l, a: d.a, b: d.b }).toHex();
        if (name) op.push({ ...d, isChecked: false, isActive: flag, hex: color });
      });
      op = Array.from(new Set(op));
      return op;
    },
  },
  created() {
    this.parseCsvData('/assets/color/firstLevel.csv', 'firstLevelList');
    this.parseCsvData('/assets/color/treeTableSecond.csv', 'secondLevelList');
    this.parseCsvData('/assets/color/secondOhterLevel.csv', 'secondOhterLevel');
    this.parseCsvData('/assets/color/thirdOtherLevel.csv', 'thirdOtherLevel');
  },
};
</script>

<style lang="scss">
.tree_container {
  width: 100%;
  height: calc(100vh - 50px);
  display: flex;
  justify-content: center;
  .content {
    max-width: 900px;
    min-width: 900px;
    height: 100%;
    .header-btn {
      display: flex;
      margin: 10px 0;
      .button {
        width: 80px;
        height: 26px;
        line-height: 26px;
        margin: 0 5px;
        border-radius: 8px;
        border: 1px solid rgb(127, 127, 127);
        color: #ddd;
        font-size: 13px;
        cursor: pointer;
        &:hover {
          color: #fff;
          border: 1px solid #fff;
        }
      }
    }
    .first-color {
      margin-top: 20px;
      .title {
        font-size: 15px;
        color: #fff;
      }
      .color-box {
        margin-top: 16px;
        display: flex;
        .color-item {
          flex: 1;
          display: flex;
          align-items: center;
          justify-content: center;
          background: #fff;
          padding: 4px 10px;
          margin: 0 10px;
          border-radius: 6px;
          position: relative;
          cursor: pointer;
          .icon {
            width: 14px;
            height: 14px;
            border-radius: 50%;
            display: flex;
            align-content: center;
            justify-content: center;
          }
          span {
            font-size: 16px;
            margin-left: 8px;
            color: #000;
          }
        }
      }
    }
    .second-color {
      margin: 20px 0;
      .title {
        display: flex;
        justify-content: space-around;
        font-size: 15px;
        color: #fff;
      }
      .color-box {
        margin-top: 16px;
        display: flex;
        .color-item {
          flex: 1;
          display: flex;
          align-items: center;
          justify-content: center;
          background: #fff;
          padding: 4px 6px;
          margin: 0 10px;
          border-radius: 6px;
          position: relative;
          cursor: pointer;
          .icon {
            width: 14px;
            height: 14px;
            border-radius: 50%;
            display: flex;
            align-content: center;
            justify-content: center;
          }
          span {
            font-size: 16px;
            margin-left: 8px;
            color: #000;
          }
        }
      }
    }
  }
  .third-color {
    .title {
      font-size: 15px;
      color: #fff;
      margin-bottom: 16px;
    }
    .third-content {
      height: calc(100vh - 266px);
      overflow: auto;
      &::-webkit-scrollbar {
        width: 10px;
      }
      &::-webkit-scrollbar-thumb {
        border-radius: 5px;
        background: rgb(49, 49, 49);
      }
      &::-webkit-scrollbar-track {
        border-radius: 0;
        background: rgb(1, 1, 1);
      }
      .color-box {
        .color-title {
          margin-left: 25px;
          margin-bottom: 10px;
          display: flex;
          align-items: center;
          .icon {
            width: 16px;
            height: 16px;
            border-radius: 50%;
          }
          span {
            color: #fff;
          }
        }
        .color-content {
          display: flex;
          flex-wrap: wrap;
          margin-bottom: 10px;
          .content-item {
            width: 140px;
            margin: 0 19px;
            .color-item {
              width: 100%;
              height: 40px;
              border-radius: 8px;
              display: flex;
              align-items: center;
              justify-content: center;
              cursor: pointer;
              &:hover {
                .hex {
                  display: block !important;
                }
              }
            }
            .color-name {
              text-align: center;
              color: #fff;
              font-size: 12px;
              margin: 5px 0;
            }
          }
        }
      }
    }
  }
  .isDisabled {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 10001; /* 遮罩层放在最上层 */
    width: 100%;
    height: 100%;
    border-radius: 6px;
    background-color: rgba(0, 0, 0, 0.5); /* 半透明黑色 */
    cursor: not-allowed;
  }
}
</style>
