<template>
    <div class="highselect"> 
        <div class="backtop">
            <img src="~assets/image/backtop.png"  v-show="topshow" v-tap="{methods:backtop}">
        </div>
        <pullrefresh :on-refresh="onRefresh" :on-infinite="onInfinite" :dataList="scrollData" v-show="show" id="scroll" @scroll.native="getScroll">
            <div class="card" v-for="(l,index) in list" :key="index">
                <p class="card-time">{{l.gmtCreate | timefilter2}}</p>
                <div class="card-content"  @click="gotoDetail(l.orderNo)">
                <!--    <img src="~assets/image/message/banner_activity.jpg" alt="">  -->
                    <p>{{l.title}}</p>
                    <div class="content">{{l.content}}</div>
                    <div class="bot" >
                        <span>查看详情</span>
                        <img src="~assets/image/arrow_l.png" alt="">
                    </div>
                </div>
            </div>
        </pullrefresh>
    </div>
</template>

<script>
import {messageListApi,teamList,getTeamList,getTeamDetail} from 'src/api'
import cookie from 'src/widget/plugin/cookie'
import pullrefresh from 'src/components/service/pullrefresh'
export default {
  name:'highselectactivity',
    data(){
        return{
            activityId:1,
            scrollTop:0,
            pageNum:1,
            pageSize:8,
            list:[],
            topshow:false,
            show:true,
            pageStart: 0, // 开始页数
            pageEnd: 0, // 结束页数
            scrollData:{
                noFlog:false //暂时没事数据
            }
        }
    },
    components:{
        pullrefresh
    },
    methods: {
          getData(){
            let userId = cookie.get('http_userID')    
            // let userId = 128
            this.axios.post(messageListApi+'/'+this.pageNum+'/'+this.pageSize,{userId:userId,type:2})
            .then(res=>{
                console.log(res)
            if(res.data.data){
                this.show = true
                if(this.list.length>0){
                    this.list = this.list.concat(res.data.data)
                }else{
                    this.list = res.data.data;
                }
            }else{
                this.show=false
            }
            this.pageEnd = res.data.total
            }).catch(err=>{
                console.log(err)
            })
          },
          gotoDetail(a) {
            let userId = cookie.get('http_userID')  
            // let userId = 128
            this.axios.post(getTeamDetail,{userId:userId,id:a}).then(res=>{
                console.log(res)
                if(res.status == 200){
                    if(res.data.data.activityTeamResultVO.activityId == 1){
                        this.$router.push({path:'/teamdetail', query:{id:a,activityId:res.data.data.activityTeamResultVO.activityId}})
                    }else{
                        this.$router.push({path:'/pkdetail', query:{id:a,activityId:res.data.data.activityTeamResultVO.activityId}})
                    }
                }
            }).catch(err=>{
                console.log(err)
            })
             
          },
          onRefresh(done){
            this.pageNum = 1;
            this.list=[];
            this.getData();
            done()
          },
          onInfinite(done){
            this.pageNum++;
            let end = Math.ceil(this.pageEnd/this.pageSize)
            // console.log(this.pageNum)
            // console.log(end)
            
            let more = this.$el.querySelector('.load-more')
            if(this.pageNum > end) {
            more.style.display = 'none'; //隐藏加载条
            //走完数据调用方法
            this.scrollData.noFlag = true;
            } else {
            this.getData()
            more.style.display = 'none'; //隐藏加载条
            }
            done()
          },
          getScroll(){
              this.scrollTop = this.$el.querySelector("#scroll").scrollTop;
              console.log(this.scrollTop)
              if(this.scrollTop>=350){
               this.topshow =true;
              }else{
                this.topshow =false;
              }
          },
          backtop(){
            let timer = setInterval(()=>{
            if(this.scrollTop > 0){
              this.$el.querySelector('#scroll').scrollTop  -= 70
            }else{
              clearInterval(timer)
              this.$el.querySelector('#scroll').scrollTop = 0
            }
            },20)
          }
        },
        created(){
            this.getData()
        },
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
@import '~src/assets/scss/function';
    .highselect{
        width:100%;
        height:100%;
        overflow-x: hidden;
        background:#f7f7f7;
        & .card{
            border:1px solid transparent;
            &>.card-time{
                width:rem-calc(85px);
                margin:22px auto 15px;
                height:rem-calc(24px);
                background:#d6d6d6;
                border-radius:rem-calc(24px);
                text-align:center;
                font-size:rem-calc(12px);
                line-height:rem-calc(24px);
                color:#ffffff;
            }
            &>.card-content{
                box-shadow: 0px 3px 20px 0px rgba(177, 177, 177, 0.16);
                margin:rem-calc(0 15px 0 15px);
                background:#ffffff;
                border-radius:rem-calc(7px);
                &>img{
                    display:block;
                    width:92%;
                    margin:rem-calc(15px) auto;
                }
                &>p{
                    padding:rem-calc(15px 10px 0 15px);
                    font-size:rem-calc(15px);
                    color:#333333;
                    margin-bottom:rem-calc(7px);
                }
                &>.content{
                    margin-left:rem-calc(15px);
                    padding:rem-calc(2px 10px 0 0);
                    font-size:rem-calc(13px);
                    color:#666666;
                    line-height:rem-calc(22px);
                    padding-bottom:rem-calc(8px);
                    border-bottom:1px solid rgb(238, 238, 238);
                }
                 &>.bot{
                    padding:rem-calc(12px 15px 16px 15px);
                    overflow:hidden;
                    &>span{
                        font-size:rem-calc(13px);
                    }
                    &>img{
                        float:right;
                        transform:rotate(180deg);
                    }
                }
            }
        }
        .backtop{
            position:absolute;
            bottom:3%;
            right:5%;
            z-index:110;
            &>img{
                width:rem-calc(65px);
                height:rem-calc(65px);
            }
        }
    }
</style>
