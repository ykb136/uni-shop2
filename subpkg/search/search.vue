<template>
    <view>
        <view class="search-box">
            <uni-search-bar  @input="input" :radius="100" cancel-button="none"></uni-search-bar>
        </view>
        <!-- 搜索建议 -->
        <view class="sugg-list" v-if="searchResults.length!==0">
            <view class="sugg-item" v-for="(item ,i) in searchResults" :key="i" @click="gotoDetail(item)">
                <view class="goods-name">{{item.goods_name}}</view>
                <uni-icons type="arrowright" size="16"></uni-icons>
            </view>
        </view>
        
        <!-- 搜索历史 -->
        <view class="history-box" v-else>
            <!-- 标题区 -->
            <view class="history-title">
                <text>搜索历史</text>
                <uni-icons type="trash" size="17" @click="clean"></uni-icons>
            </view>
            <!-- 列表区 -->
            <view class="history-list">
                <uni-tag :text="item" v-for="(item,i) in histories" :key="i" @click="gotoGoodsList(item)"></uni-tag>
            </view>
        </view>
    </view>
</template>

<script>
    export default {
        methods:{
            //input输入事件的处理函数
            input(e){
                // console.log(e)
                clearTimeout(this.timer)
            this.timer=setTimeout(()=>{
                    // console.log(e)
                    this.kw=e,
                    this.getSearchList()
                },500)
            },
        async  getSearchList(){
                //判断为空
                if(this.kw.length===0){
                    this.searchResults=[]
                    return
                }
      const {data:res} =  await  uni.request({
                    url:'https://api-hmugo-web.itheima.net/api/public/v1/goods/qsearch',
                    data:{query:this.kw},
                    method:'GET'
                })
        if(res.meta.status!==200) return uni.showToast({
            title:"请求失败！",
            duration:1500,
            icon:'none'
        })
        this.searchResults=res.message
        // console.log(this.searchResults)
        this.saveSearchHistory()
            },
            gotoDetail(item){
                // console.log(item.goods_id)
                uni.navigateTo({
                    url:'/subpkg/goods_detail/goods_detail?goods_id='+item.goods_id
                })
            },
            saveSearchHistory(){
                // this.historyList.push(this.kw)
                const set= new Set(this.historyList)
                set.delete(this.kw)
                set.add(this.kw)
                this.historyList=Array.from(set)
                
                //对搜索进行持久化存储
                uni.setStorageSync('kw',JSON.stringify(this.historyList))
            },
            clean(){
                this.historyList=[]
                uni.setStorageSync('kw','[]')
            },
            gotoGoodsList(kw){
                uni.navigateTo({
                    url:'/subpkg/goods_list/goods_list?query='+kw
                })
            }
        },
        computed:{
            histories(){
                return [...this.historyList].reverse()
            }
        },
        data() {
            return {
                //延时器
                timer:null,
                kw:'',
                searchResults:[],
                //搜索历史数组
                historyList:[]
            };
        },
        onLoad() {
         this.historyList= JSON.parse(uni.getStorageSync('kw')|| '[]') 
        },
    }
</script>

<style lang="scss">
.search-box{
    position: sticky;
    top:0;
    z-index: 999;
}
.sugg-list{
    padding: 0 5px;
    .sugg-item{
        display: flex;
        align-items: center;
        justify-content: space-between;
        font-size: 12px;
        padding: 13px 0;
        border-bottom: 1px solid #efefef;
        .goods-name{
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }
    }
}
.history-box{
    padding: 0 5px;
   .history-title{
       display: flex;
       justify-content: space-between;
       height: 40px;
       align-items: center;
       font-size: 13px;
       border-bottom: 1px solid #efefef;
   } 
   .history-list{
       display: flex;
       flex-wrap: wrap;
       .uni-tag{
           margin-top: 5px;
           margin-right: 5px;
       }
   }
}
</style>