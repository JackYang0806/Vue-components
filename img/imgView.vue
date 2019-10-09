<template>
<!-- 使用方法 

import imgView from './imgView'
 
    <imgView :imglist="viewImgList" :showIndex="viewIndex" :state="viewState" @changeState="viewStatechange"></imgView>

 data:{
     viewImgList:[
        {
            url:"",
            name:""
        }
     ],
     viewState：false,  //控制预览控件显隐
     viewIndex:0,  //当前展示图片序号
 },
 methods:{
     changeState(){
         //接收预览控件的关闭事件   置空条件
         this.viewState=false;
         this.viewIndex=0;
         this.viewImgList=[];
     }
 }
 组件  做了阻止事件冒泡和捕获   防止浏览器点击图片新建窗口   
-->
    <div class="img-view-components" v-if="imglist.length>0" v-show="state" @mousewheel="stopEvent($event)" @DOMMouseScroll="stopEvent($event)">
        <!-- 滚动事件兼容  IE Chrome mousewheel   FireFox DOMMouseScroll -->
        <h4>{{imglist[activeIndex].name}} <i class="el-icon-close" @click="closeView"></i></h4>
        <div class="img-view-main" v-loading="loading" ref="imgBox" :style="{height:boxHegiht+'px'}" @mousedown="start($event)" @mousemove="move($event)" @mouseleave="end($event)" @mouseup="end($event)" @mousewheel="stopEvent($event)" @DOMMouseScroll="stopEvent($event)">
            <img @dragstart="stopEvent($event)" @DOMMouseScroll="scrollScale($event)" @mousewheel="scrollScale($event)" draggable='false' @load="imgReady()"   ref="imgDom" :style="{transform:'rotate('+styles.rotate+'deg) scale('+styles.scale+')',marginLeft:styles.left+'px',marginTop:styles.top+'px'}" :src="imglist[activeIndex].url">
            <!-- <div class="eventsBox" @mousedown="start($event)" @mousemove="move($event)" @mouseup="end($event)" @click="showInfos"></div> -->
        </div>
        <span class="arrow arrow-left" v-show="imglist.length>1" @click="imgChange(false)"><i class="el-icon-arrow-left"></i></span>
        <span class="arrow arrow-right" v-show="imglist.length>1" @click="imgChange(true)"><i class="el-icon-arrow-right"></i></span>
        
        <div class="options-main">
            <i class="el-icon-zoom-out" @click="scaleClick(false)"></i>
            <i class="el-icon-zoom-in" @click="scaleClick(true)"></i>
            <!-- <i class="el-icon-refresh" @click="imgRotate"></i> -->
            <span @click="rotateclick(false)"><icon-svg name="rotate-left"  class="icon-rotate icon-rotate-left"></icon-svg></span>
            <span @click="rotateclick(true)"><icon-svg name="rotate-right"  class="icon-rotate icon-rotate-right"></icon-svg></span>
            
            
        </div>
    </div>
</template>
<script>
export default {
    data() {
        return {
            loading:true,
            activeIndex:0,
            // activeUrl:"https://www.ouralt.com/group1/M00/00/06/CgAYKl1D25eAZEU9AAqS3LYdu7g781.png",
            // activeName:"测试标题",
            styles:{
                rotate:0,
                left:0,
                top:0,
                scale:1,
                // naturalWidth:0,
                // naturalHeight:0,
                // containWidth:0,
                // containHeight:0,
            },
            boxHegiht:0,
            scrollState:{
                change:false,
                x:0,
                y:0,
                moveX:0,
                moveY:0,
            }
        }
    },
    props:{
        // 控制预览组件 显隐
        state:{
            default:true,
            type:Boolean
        },
        // 预览 图片数据数据   数据格式为{ url:"xxxx",name:"xxxx"}  
        imglist:{
            type:Array,
            default:[]
        },
        // 预览序号
        showIndex:{
            type:Number,
            default:0
        }
    },
    watch: {
        state(val){
            let dom=document.getElementsByTagName('body')[0];
            if(val){
                dom.style.height='100%';
                dom.style.overflow='hidden';
            }
            else{
                dom.style.height='auto';
                dom.style.overflow='visible';
            }
        },
        showIndex(val){
            this.loading=true;
            this.activeIndex=val;
        }
    },
    created() {
        let _this=this;
        let h=document.documentElement.clientHeight;//可视区域高度
        _this.boxHegiht=h-120;
        // window.onresize=function(){
        //     let h=document.documentElement.clientHeight;//可视区域高度
        //     _this.boxHegiht=h-120;
        // }
    },
    methods: {
        closeView(){
            this.$emit("changeState",false);
            this.activeIndex=0;
        },
        imgChange(type){
            let index=type?1:-1;
            index=this.activeIndex+index;
            if(index<0){
                index=this.imglist.length-1;
            }
            if(index>=this.imglist.length){
                index=0;
            }
            this.activeIndex=index;
            this.loading=true;
        },
        // 图片加载完成后
        imgReady(ev){
            this.loading=false;
            this.$set(this.styles,'rotate',0)
            this.$set(this.styles,'left',0)
            this.$set(this.styles,'top',0)
            this.$set(this.styles,'scale',1)

            // // 图片原始尺寸
            // this.styles.naturalWidth=this.$refs['imgDom'].naturalWidth;
            // this.styles.naturalHeight=this.$refs['imgDom'].naturalHeight;
            // // 图片当前显示尺寸
            // this.styles.containWidth=this.$refs['imgDom'].width;
            // this.styles.containHeight=this.$refs['imgDom'].height;
        },
        // 旋转
        rotateclick(type){
            if(this.loading){
                return false
            }
            let result=type?90:-90;
             result=this.styles.rotate+result;
            this.$set(this.styles,'rotate',result)
        },
        start(ev){
            if(this.loading){
                return false
            }
            this.scrollState.change=true;
            ev.stopPropagation();
            ev.stopPropagation();
        },
        move(ev){
            if(this.loading){
                return false
            }
            ev.stopPropagation();
            ev.stopPropagation();
            if(!this.scrollState.change){
                return false
            }
            // 第一次点击  不移动 
            if(this.scrollState.moveX==0){
                this.scrollState.moveX=ev.pageX;
                this.scrollState.moveY=ev.pageY;
                return false
            }
            let x=ev.pageX-this.scrollState.moveX;
            let y=ev.pageY-this.scrollState.moveY;
            this.scrollState.moveX=ev.pageX;
            this.scrollState.moveY=ev.pageY;
                this.styles.left+=x;
                this.styles.top+=y;

        },
         end(ev){
             if(this.loading){
                return false
            }
               ev.stopPropagation();
            ev.stopPropagation();
             if(!this.scrollState.change){
                 return false;
             }
            //  重置移动
            this.scrollState.change=false;
            this.scrollState.moveX=0;
            this.scrollState.moveY=0;
        },
        // 放大缩小
        scrollScale(ev){
            if(this.loading){
                return false
            }
            ev.stopPropagation();
            ev.stopPropagation();
            let scale=this.styles.scale;
            //  IE  chrome   wheelDelta:120 ******FireFox  detail -3  上滑    
            if(ev.wheelDelta>0||ev.detail<0){
                scale+=0.05;
            }
            else{
                scale=scale-0.05;
            }
            this.$set(this.styles,'scale',scale)

        },
        // 点击缩放
        scaleClick(type){
            if(this.loading){
                return false
            }
            let scale=type?0.05:-0.05;
             scale=this.styles.scale+scale;
            this.$set(this.styles,'scale',scale)
        },
        stopEvent(ev){
            ev.preventDefault();
            ev.stopPropagation();
            return false;
        }
    }
}
</script>
<style lang="scss">
    .img-view-components{
        position: fixed;
        z-index: 999999;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        background: rgba(20,20,20,0.8);
        h4{
           text-align: center;
           font-size: 16px;
           line-height: 20px;
           color: #fff;
           padding: 20px;
           i{
               float: right;
               cursor: pointer;
               border: solid 1px #fff;
                border-radius: 50%;
                padding: 4px;
                &:hover{
                    color: #1b69a6;
                    border-color: #1b69a6;
                }
           }
        }
        .img-view-main{
            position: relative;
            height: 80%;
            overflow: hidden;
            width: 1200px;
            margin: 0 auto;
            border:  solid 1px #dadada;
            display: flex;
            align-items: center;
            justify-content: center;
            img{
                max-width: 100%;
                max-height: 100%;
                transform-origin: center center;
                -webkit-tap-highlight-color: transparent;
                user-select: none;
            }
            .eventsBox{
                position: absolute;
                left: 0;
                right: 0;
                top: 0;
                bottom: 0;
            }
        }
        .arrow{
            font-size: 20px;
            width: 30px;
            height: 30px;
            border: solid 1px #fff;
            border-radius: 50%;
            text-align: center;
            line-height: 30px;
            position: absolute;
            top: 48%;
            color: #fff;
            transform: translateY(-50%);
            &.arrow-left{
                left: 30px;
            }
            &.arrow-right{
                right: 30px;
            }
            &:hover{
                border-color: #1b69a6;
                color: #1b69a6;
            }
        }
        .options-main{
            position: absolute;
            left: 0;
            right: 0;
            bottom: 20px;
            font-size: 20px;
            line-height: 20px;
            color: #fff;
            text-align: center;
            user-select: none;
            i{
                cursor: pointer;
                margin: 0 10px;
                vertical-align: top;
            }
            .icon-rotate{
                vertical-align: top;
                width: 20px;
                height: 20px;
                margin: 0 10px;
                cursor: pointer;
            }
        }
    }
</style>