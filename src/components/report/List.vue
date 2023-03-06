<template>
  <div class="com-container">
    <div class="list-wrapper">
      <div class="title" :style="comStyle">
        <span class="before-icon">▎</span>
        <span>条件查询</span>
      </div>
      <div class="search-container">
        <el-input v-model="searchContent" placeholder="请输入地址" @input="search" />
        <el-date-picker v-model="searchDate" type="daterange" range-separator="-" start-placeholder="起始日期" end-placeholder="结束日期" @change="search" />
      </div>
      <div class="component">
        <div ref="forwardScrollBox" class="list_dv" @mouseenter="mouseenter" @mouseleave="mouseleave">
          <div ref="forwardContent" class="scroll_dv">
            <div v-for="(item, index) in list" :key="index" class="list_item">
              <div class="ico_dv dfcen">
                <img
                  :src="`https://desk-calenda.oss-cn-beijing.aliyuncs.com/static-h5/taili-ms/kanban/forward_tit_${index < 3 ? index + 1 : 4}.png`"
                  class="num_ico"
                />
                <span>{{ index + 1 }}</span>
              </div>
              <div class="text_line1">{{ item.address }}</div>
              <div class="statistic_dv">
                <img
                  v-for="numItem in Math.floor(((item.number / totalCount) * 100) / 5)"
                  :key="numItem + 100"
                  :src="`https://desk-calenda.oss-cn-beijing.aliyuncs.com/static-h5/taili-ms/kanban/forward_base_${index < 3 ? index + 1 : 4}.png`"
                  class="forward_base_ico"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import axios from 'axios'
import { mapState } from 'vuex'
import { getThemeValue } from 'utils/theme_utils'

export default {
  components: {},
  props: {},
  data() {
    return {
      axiosInstance: null,
      searchContent: null,
      searchDate: null,
      totalCount: 10000, // 总数
      scrollContentHeight: 0, // 滚动内容的高度
      scrollBoxHeight: 0, // 滚动父级盒子的高度
      scrollableHeight: 0, // 可以滚动的高度
      scrollTop: 0, // 元素已经滚动的高度
      timer2: null, // 定时器
      list: []
    }
  },
  computed: {
    ...mapState(['theme']),
    // 设置给标题的样式
    comStyle() {
      return {
        fontSize: this.titleFontSize + 'px',
        color: getThemeValue(this.theme).titleColor,
      }
    },
  },
  watch: {},
  created() {
    this.axiosInstance = axios.create({
      baseURL: 'http://127.0.0.1:9997',
    })
  },
  mounted() {
    this.fetchData()
  },
  destroyed() {
    // 离开时销毁定时器
    clearInterval(this.timer2)
  },
  methods: {
    async fetchData() {
      const { data: list } = await this.axiosInstance.get('/project')
      if(list && list.length > 0 ){
        this.list = list
        this.generateMarkers(list)
        this.regainHeight()
        this.$nextTick(() => {
          // 窗口大小变化时，重新获取滚动相关元素的高度
          window.onresize = this.regainHeight()
          // 监听元素滚动事件
          // this.$refs.forwardScrollBox.addEventListener('scroll', this.handleScroll, true)
        })
      }
    },
    async search() {
      if(this.searchContent && this.searchDate){
        console.log('search');
        const { data:list } = await this.axiosInstance.post('/project', {
          searchContent: this.searchContent,
          searchDate: this.searchDate
        })
        this.list = list
        this.generateMarkers(list)
      }
    },
    generateMarkers(list) {
      let markers = []
      list.forEach(item => {
        let lng = item.lnglat.split(',')[0]
        let lat = item.lnglat.split(',')[1]
        let marker = {
          lng,
          lat,
          title: item.address
        }
        markers.push(marker)
      })
      this.$store.commit('mark', {markers})
    },
    // 页面第一次进入或者窗口大小变化时，获取滚动相关元素的高度
    regainHeight() {
      // 获取需要滚动元素的真实高度
      this.scrollContentHeight = this.$refs.forwardContent.getBoundingClientRect().height
      // 获取限制高度滚动容器的真实高度
      this.scrollBoxHeight = this.$refs.forwardScrollBox.getBoundingClientRect().height
      // 可以滚动的高度
      this.scrollableHeight = this.scrollContentHeight - this.scrollBoxHeight
      console.log('内容高度', this.scrollContentHeight, '盒子高度', this.scrollBoxHeight, '可以滚动的高度', this.scrollableHeight)
      this.autoScroll()
    },
    // 自动滚动
    autoScroll() {
      this.timer2 = setInterval(() => {
        if (this.scrollableHeight >= this.scrollTop) {
          // 当为滚动到结尾时滚动
          this.scrollTop = this.$refs.forwardScrollBox.scrollTop += 1
        } else {
          // 滚动结束，清除当前定时器，scrollTop重置为0，继续滚动
          this.scrollTop = this.$refs.forwardScrollBox.scrollTop = 0
          clearInterval(this.timer2)
          this.autoScroll()
        }
        console.log('转发数定时器')
      }, 100)
    },
    // 鼠标移入时停止滚动
    mouseenter() {
      clearInterval(this.timer2)
    },
    // 鼠标移开继续滚动
    mouseleave() {
      this.autoScroll()
    },
    // 滚动元素滚动时的方法
    handleScroll(e) {
      console.log(e.target.scrollTop)
    }
  }
}
</script>
<style lang="less" scoped>
.list-wrapper {
  width: 100%;
  height: 100%;
  background-color: rgba(171, 239, 239, 0.08);
  border-radius: 20px;
  padding: 20px 26px;
  box-sizing: border-box;
  .title {
    margin-bottom: 10px;
  }
  .search-container {
    margin-bottom: 10px;
    /deep/ .el-input__inner {
      background-color: rgba(0, 0, 0, 0);
      border: none;
    }
    /deep/ input {
      background-color: rgba(0, 0, 0, 0);
      border: none;
    }
  }
}
.text_line1 {
  margin-right: 10px;
  width: 32%;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.component {
  height: 100%;
  font-family: Source Han Sans CN, Source Han Sans CN-Regular;
  letter-spacing: 1px;
  .list_dv::-webkit-scrollbar {
    display: none;
  }
  .list_dv {
    width: 100%;
    height: 100%;
    overflow-y: auto;
    font-size: 14px;
    color: #d4eaf6;
    overflow: hidden;

    .list_item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      width: 100%;
      height: 26px;
      margin-top: 11px;
      margin-bottom: 11px;
      .ico_dv {
        display: flex;
        width: 26px;
        height: 26px;
        margin-right: 16px;
        position: relative;
        .num_ico {
          width: 100%;
          height: 100%;
          position: absolute;
          left: 0;
          top: 0;
          border-radius: 50%;
        }
        span {
          width: 26px;
          height: 26px;
          position: relative;
          line-height: 26px;
          text-align: center;
        }
      }
      h6 {
        width: 110px;
        height: 26px;
        margin-right: 10px;
        line-height: 26px;
      }
      .statistic_dv {
        width: 50%;
        height: 18px;
        overflow: hidden;
        .forward_base_ico {
          width: 6px;
          height: 18px;
          margin-right: 4px;
        }
        .forward_bg_ico {
          width: 6px;
          height: 18px;
          margin-right: 4px;
          opacity: 0.5;
          background: #164482;
          border-radius: 2px;
        }
      }
      .current_num {
        width: 48px;
        height: 26px;
        font-size: 16px;
        line-height: 26px;
      }
    }
  }
}
</style>
