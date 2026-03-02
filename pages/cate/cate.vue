<template>
    <view>
        <!-- 使用自定义搜索组件 -->
        <!-- <my-search :bgcolor="'pink'" :radius="3"></my-search> -->
        <my-search @click="gotoSearch"></my-search>
    <view class="scroll-view-container">
        <!-- 左滑动 -->
        <scroll-view scroll-y="true" :style="{height: wh+'px'}" class="left-scroll-view">
            <block v-for="(item,i) in cateList" :key="i">
                <view :class="['left-scroll-view-item',i===active?'active':' ']" @click="activeChanged(i)">{{item.cat_name}}</view>
            </block>
            
        </scroll-view>
       <!-- 右滑动 -->
       <scroll-view scroll-y="true" :style="{height: wh+'px'}" :scroll-top="scrollTop">
           <view class="cate-lv2" v-for="(item2,i2) in cateLevel2" :key="i2">
               <!-- 二级分类标题 -->
               <view class="cate-lv2-title">
                   /  {{item2.cat_name}}  /
               </view>
               <!-- 2级下的3级列表 -->
               <view class="cate-lv3-list">
                   <view class="cate-lv3-item" v-for="(item3,i3) in item2.children" :key="i3" @click="gotogoodsList(item3)">
                       <image :src="item3.cat_icon"></image>
                       <text>{{item3.cat_name}}</text>
                   </view>
               </view>
           </view>
        
       </scroll-view>
    </view>
    </view>
</template>

<script>
    export default {
        data() {
            return {
                cateList:[],
               //当前设备可用高度
                wh:0,
                active:0,
                cateLevel2:[],
                scrollTop:0
            };
        },
        onLoad(){
            const sysInfo=uni.getSystemInfoSync()
            console.log(sysInfo)
            this.wh=sysInfo.windowHeight-50
            this.getCateList()
        },
        methods:{
            //获取分类列表的数据
         async getCateList(){
           const {data:res}=await uni.request({
                    url:"https://api-hmugo-web.itheima.net/api/public/v1/categories",
                    method:'GET'
                })
                if (res.meta.status !==200) return uni.showToast({
                    title:"数据加载失败",
                    duration:1500,
                    icon:'none'
                })
                this.cateList=res.message
                //2级
                this.cateLevel2=res.message[0].children
                console.log(this.cateLevel2)
            },
            activeChanged(i){
                this.active=i
                //重新为2级赋值
                this.cateLevel2=this.cateList[i].children
                this.scrollTop=this.scrollTop===0?1:0
            }, //跳转到商品列表页面
            gotogoodsList(item){
                uni.navigateTo({
                    url:'/subpkg/goods_list/goods_list?cid='+item.cat_id
                })
            },
            gotoSearch(){
                uni.navigateTo({
                    url:'/subpkg/search/search'
                })
            }
        }
    }

</script>

<style lang="scss">
    
.scroll-view-container{
    display: flex;
    .left-scroll-view{
    width: 120px;
    .left-scroll-view-item{
        background-color: #F7F7F7;
        line-height: 60px;
        text-align: center;
        font-size: 12px;
        
        &.active{
            background-color: #FFFFFF;
            position: relative;
            
            &::before{
                content: ' ';
                display: block;
                width: 3px;
                height: 30px;
                background-color: #c00000;
                position: absolute;
                top: 50%;
                left: 0;
                transform: translateY(-50%);
            }
        }
    }
    }
}
.cate-lv2-title{
    font-size: 12px;
    font-weight: bold;
    text-align: center;
    padding: 15px 0;
}
.cate-lv3-list{
    display: flex;
    flex-wrap: wrap;
   .cate-lv3-item{
       width: 33.33%;
       display: flex;
       flex-direction: column;
       justify-content: center;
       align-items: center;
       margin-bottom: 10px;
       image{width: 60px; height: 60px;}
       text{font-size: 12px;}
   } 
}
</style>