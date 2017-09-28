<template>
  <div id="app">
    <view-box>
      <x-header 
        slot="header"
        style="width:100%;position:absolute;left:0;top:0;z-index:100;"
        title="网易" 
        :right-options="{showMore:true}"
        :left-options="{showBack:true,backText:'返回',preventGoBack:true}"
      ></x-header>


      <sc
        :lock-y="true"
        :scrollbar-x="true"
      >
        <div class="tab">
          <tab>
             <!-- <tab-item selected>{{barlist[0]}}</tab-item> -->
            <tab-item v-for="intem ,index in barlist" :key="index" :selected="index==0?true:false">{{intem}}</tab-item>
          </tab>
        </div>
      </sc>

    <scroller 
      class="my"
      :on-refresh="refresh"
      :on-infinite="infinite"
      refresh-text="正在加载"
      no-data-text="没有更多数据"
      ref="mrfresh"
      >

      <swiper
        :list="list"
        v-model="swiperIndex"
        :loop="true"
      ></swiper>


      <marquee>
          <marquee-item v-for="item ,index in marqueeList" :key="index">{{item.title}}</marquee-item>
      </marquee>


      <panel 
        header="图文组合列表" 
        :list="datalist" 
        >
      </panel>
        <!-- :footer="footer"  -->

    </scroller>
      
      <tabbar slot="bottom">
        <tabbar-item>
          <i slot="icon" class="iconfont">&#xe681;</i>
          <span slot="label">首页</span>
        </tabbar-item>
        <tabbar-item show-dot>
          <i slot="icon" class="iconfont">&#xeba7</i>
          <span slot="label">短信</span>
        </tabbar-item>
        <tabbar-item selected link="/component/demo">
          <i slot="icon" class="iconfont">&#xe641;</i>
          <span slot="label">微信</span>
        </tabbar-item>
        <tabbar-item badge="8">
          <i slot="icon" class="iconfont">&#xe651;</i>
          <span slot="label">QQ</span>
        </tabbar-item>
      </tabbar>
    </view-box>
  </div>
</template>

<script>
var refreshkey = ['A','B01','B02','B03','B04','B05','B06','B07','B08','BO9','B010']
var key = -1 ;
var start = 0 ;
var end = start + 9 ;
function getKey(){
  key++;
  return refreshkey[key%11]
}

//导入vux
import {Panel, Marquee, MarqueeItem ,Swiper,Scroller as sc,Tab, TabItem,ViewBox, XHeader, Actionsheet, TransferDom, ButtonTab, ButtonTabItem, Tabbar, TabbarItem, Group, Cell } from 'vux'
export default {
  created(){
    this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html').then(data => {
      console.log(data);
      var list = [],datalist=[],marqueeList=[];
      data.focus.forEach(function(item,index) {
        if(!item.addata){
          list.push({
            url: item.link,
            img: item.picInfo[0].url,
            title: item.title
          })
        }
      });
      this.list = list;

      data.list.forEach(function(item,index) {
        if(!item.addata){
          datalist.push({
            src: item.picInfo[0].url,
            title: item.title,
            desc: item.digest || item.source,
            url: item.link
          })
        }
      });
      this.datalist = datalist;
      start = 0;
      end = start + 9 ;

      data.news.forEach(function(item,index) {
        if(item.category && item.digest){
          marqueeList.push({
            category: item.category,
            digest: item.digest,
            link: item.link,
            title:item.title,
            index:index
          })
        }
      });
      this.marqueeList = marqueeList;
    });
  },
  mounted(){
    var len = this.barlist.length;
    var otab = document.getElementsByClassName('tab')[0];
    otab.style.width = 50*len + '%';
  },
  methods:{

    refresh:function(){
      this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html',{
        miss:'00',
        refresh:getKey()
      }).then(data => {
        var datalist = [];
        data.list.forEach(function(item,index) {
          if(!item.addata){
            datalist.push({
              src: item.picInfo[0].url,
              title: item.title,
              desc: item.digest || item.source,
              url: item.link
            })
          }
        });
        this.datalist = datalist;
        this.$refs.mrfresh.finishPullToRefresh();
        start = 0;
        end = start + 9 ;
      });
    },
    infinite:function(){
      start += 10;
      end = start + 9 ;
      console.log(start,end);
      this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/10-19.html',{
        miss:'00',
        refresh:'B01'
      }).then(data => {
        console.log('uploading');
        console.log(data);
        var _this = this ;
        setTimeout(function(){
          var datalists = [];
          data.list.forEach(function(item,index) {
            if(!item.addata &&　item.picInfo.length){
              datalists.push({
                src: item.picInfo[0].url,
                title: item.title,
                desc: item.digest || item.source,
                url: item.link
              })
            }
          });
          _this.datalist = _this.datalist.concat(datalists);
          _this.$refs.mrfresh.finishInfinite();
        },1000);
      });
    },
  },
  name: 'app',
  data(){
    return{
      list:[],
      datalist:[],
      marqueeList:[],
      footer: {
        title: '查看更多',
      },
      swiperIndex:0,
      barlist:['新闻','推荐','国际','QQ','中国']
    }
  },
  components:{
  Panel,Marquee, MarqueeItem ,Swiper,sc, Tab, TabItem,ViewBox, XHeader, Actionsheet, TransferDom, ButtonTab, ButtonTabItem,Tabbar, TabbarItem, Group, Cell
  }
}
</script>

<style lang="less">
html,body{
  a{
    text-decoration: none;
  }
  width: 100%;
  height: 100%;
  overflow-x: hidden;
}
#app {
  height: 100%;
  overflow: hidden;
  .tab{
    margin-top:45px;
    // width: 300%;
  }
  .my{
    top:90px;
  }
  .weui-tabbar{
    padding-top:10px;
  }
  .weui-tabbar__label{
    margin-top:0;
  }
  .loading-layer{
    padding-bottom: 130px;
  }
}
</style>
