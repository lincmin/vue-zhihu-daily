<template>
  <div id="app">
    <div class="daily">
      <div class="daily-menu">
        <div class="daily-menu-item"
             :class="{on:type==='recommend'}"
             @click="handleToRecommend">每日推荐</div>
        <div class="daily-menu-item"
             :class="{on:type==='daily'}"
             @click="showThemes = !showThemes">主题日报</div>
        <ul v-show="showThemes">
          <li v-for="(item,index) in themes"
              :key="index">
            <a href=""
               :class="{on:item.id===themeId && type==='daily'}"
               @click.prevent="handleToTheme(item.id)">{{item.name}}</a>
          </li>
        </ul>
      </div>
      <div class="daily-list">
        <template v-if="type==='recommend'">
          <div v-for="(list,index) in recommendList"
               :key="index">
            <div class="daily-date">{{formatDay(list.date)}}</div>
            <Item v-for="item in list.stories"
                  :data="item"
                  :key="item.id"></Item>
          </div>
        </template>
        <template v-if="type==='daily'">
          <Item v-for="item in list"
                :data="item"
                :key="item.id"></Item>
        </template>
      </div>

    </div>
  </div>
</template>

<script>
import Item from './components/item.vue'
import $ from './libs/util.js'
import axios from 'axios'

export default {
  name: 'app',
  data() {
    return {
      themes: [],
      showThemes: false,
      type: 'recommend',
      themeId: 0,
      isLoading: false,
      dailyTime: $.getTodayTime(),
      recommendList: [],
      list: []
    }
  },
  components: { Item },
  methods: {
    //请求主题
    getThemes() {
      $.ajax.get('themes').then(res => {
        this.themes = res.others
      })
    },
    handleToTheme(id) {
      //改变菜单分类
      this.type = 'daily'
      //设置当前点击子类的主题日报id
      this.themeId = id
      //清空中间栏数据
      this.list = []
      //获取主题下的文章列表
      $.ajax.get('theme/' + id).then(res => {
        //过滤类型为1的文章，该类型下的文章为空
        this.list = res.stories.filter(item => item.type != 1)
      })
    },
    handleToRecommend() {
      this.type = 'recommend'
      this.recommendList = []
      this.dailyTime = $.getTodayTime()
      this.getRecommendList()
    },
    getRecommendList() {
      //获取每日推荐数据
      this.isLoading = true
      const prevDay = $.prevDay(this.dailyTime + 86400000)
      $.ajax.get('news/before/' + prevDay).then(res => {
        this.recommendList.push(res)
        this.isLoading = false
      })
    },
    formatDay(date) {
      //转换为带汉字的月日
      let month = date.substr(4, 2);
      let day = date.substr(6, 2);
      if (month.substr(0, 1) === '0') month = month.substr(1, 1);
      if (day.substr(0, 1) === '0') day = day.substr(1, 1);
      return `${month} 月 ${day} 日`;
    }
  },
  mounted() {
    //初始化调用
    this.getThemes()
    this.getRecommendList()
  }
}
</script>

<style>

</style>
