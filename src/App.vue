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
      <div class="daily-list"
           ref="list"
           @scroll="handleScroll">
        <template v-if="type==='recommend'">
          <div v-for="(list,index) in recommendList"
               :key="index">
            <div class="daily-date">{{list.date}}</div>
            <Item v-for="item in list.stories"
                  :data="item"
                  :key="item.id"
                  @click.native.prevent="handleClick(item.id)"></Item>
          </div>
        </template>
        <template v-if="type==='daily'">
          <Item v-for="item in list"
                :data="item"
                :key="item.id"
                @click.native.prevent="handleClick(item.id)"></Item>
        </template>
      </div>
      <daily-article :id="articleId"></daily-article>
    </div>
  </div>
</template>

<script>
import Item from './components/item.vue'
import dailyArticle from './components/daily-article.vue'
import $ from './libs/util.js'
import axios from 'axios'
import moment from 'moment'

export default {
  name: 'app',
  data() {
    return {
      themes: [],
      showThemes: false,
      type: 'recommend',
      themeId: 0,
      isLoading: false,
      dailyTime: moment().unix(),
      recommendList: [],
      list: [],
      articleId: 0
    }
  },
  components: { Item, dailyArticle },
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
      this.dailyTime = moment().unix()
      this.getRecommendList()
    },
    getRecommendList() {
      //获取每日推荐数据
      this.isLoading = true
      const prevDay = moment.unix(this.dailyTime).add(1, 'days').format('YYYYMMDD')
      $.ajax.get('news/before/' + prevDay).then(res => {
        res.date = res.date.replace(/(\d{4})(\d{2})(\d{2})/g, '$1年$2月$3日')
        this.recommendList.push(res)
        this.isLoading = false
      })
    },
    handleScroll() {
      //获取DOM
      const $list = this.$refs.list
      //在‘主题日报’或者在加载推荐列表时停止操作
      if (this.type === 'daily' || this.isLoading) return
      //滚动距离+页面高度=内容区域，则接触底部
      if ($list.scrollTop + document.body.clientHeight >= $list.scrollHeight) {
        //时间减少一天
        this.dailyTime = moment.unix(this.dailyTime).subtract(1, 'days').unix()
        this.getRecommendList()
      }
    },
    handleClick(id) {
      this.articleId = id
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
