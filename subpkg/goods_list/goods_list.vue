<template>
    <view>
        <view class="goods-list">
            <view v-for="(goods,i) in goodsList" :key="i" @click="gotoDetail(goods)">
                <my-goods :goods="goods"></my-goods>
            </view>
        </view>
    </view>
</template>

<script>
    export default {
        data() {
            return {
                //请求参数对象
                queryObj: {
                    query: '',
                    cid: '',
                    pagenume: 1,
                    pagesize: 10,
                    isloading:false
                },
                goodsList: [],
                total: 0,
                
            };
        },
        onLoad(options) {

            this.queryObj.query = options.query || ''
            this.queryObj.cid = options.cid || ''
            // console.log(this.queryObj)

            this.getGoodsList()
        },
        methods: {
            //获取商品列表数据方法
            async getGoodsList(cb) {
                this.isloading=true
                const {
                    data: res
                } = await uni.request({
                    url: "https://api-hmugo-web.itheima.net/api/public/v1/goods/search",
                    method: 'GET',
                    data: this.queryObj
                })
                if (res.meta.status !== 200) return uni.showToast({
                    title: "数据请求失败",
                    duration: 1500,
                    icon: 'none'
                })
                this.isloading=false
                cb &&cb()
                this.goodsList = [...this.goodsList,...res.message.goods]
                this.total = res.message.total
                console.log(this.goodsList)
                
            },
            gotoDetail(goods){
                uni.navigateTo({
                    url:'/subpkg/goods_detail/goods_detail?goods_id=' +goods.goods_id
                })
            }
        },
        onReachBottom(){
            if(this.queryObj.pagenume*this.queryObj.pagesize>=this.total) return uni.showToast({
                title:"数据加载完毕",
                duration:1500,
                icon:'none'
            })
            if(this.isloading) return 
            //让页码值自增+1
            this.queryObj.pagenume++
            this.getGoodsList()
        },
        onPullDownRefresh(){
            this.queryObj.pagenume=1
            this.total=0
            this.isloading=false
            this.goodsList=[]
            this.getGoodsList(()=>uni.stopPullDownRefresh())
        }
    }
</script>

<style lang="scss">
    
</style>