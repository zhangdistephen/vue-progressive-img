<template>
    <div class="progressive" :style="width&&height?{maxWidth:width+'px',maxHeight:height+'px'}:{}">
        <div class='progressive-fill' :style="{paddingBottom:paddingBottom+'%'}"></div>
        <div class="progressive-media">
            <img class='progressive-img' @load='handlePlaceholderLoad' ref='placeholder' v-show='false' :src="placeholder" crossorigin="anonymous" />
            <canvas  class='progressive-img' :style="{width:maxWidth+'px',height:maxHeight+'px'}" ref="canvas"></canvas>
        	<transition name="fade">
        	    <img class="progressive-img" v-show='isLoaded' @load="handleImgLoad" ref="img" :data-src="src"/>
            </transition>
        </div>
    </div>
</template>
<script type="text/babel">
    import StackBlur from "../lib/stackblur"
    const listeners=[];
    const imgCache=[];
    const handleScroll=()=> {
        listeners.forEach((el)=> {
            if(imgCache.indexOf(el)>-1) return;
            let rect = el.$refs.canvas.getBoundingClientRect()
            let [top,bottom,left,right]=[rect.top, rect.bottom, rect.left, rect.right];
            console.log([top,bottom],window.innerHeight)
            if (top < window.innerHeight && bottom > 0 && left < window.innerWidth && right > 0) {
                el.$refs.img && (el.$refs.img.src = el.$refs.img.dataset.src)
                imgCache.push(el)
                listeners.remove(el)
            }
        })
    }
    const throttle=(func,delay)=> {
        let now, last, delta, timeout
        return function () {
            if (timeout) return;
            now= +new Date()
            delta = last ? delay - (now - last) : 0
            const args = arguments;
            const context = this;
            const runCallback = function () {
                timeout = null
                last = +new Date()
                func.apply(context, args)
            }
            if (delta <= 0) {
                runCallback()
            } else {
                timeout = setTimeout(runCallback, delta)
            }
        }

    }
    const lazyLoad=throttle(handleScroll,300)
    const EVENTS = ['scroll', 'wheel', 'mousewheel', 'resize']
    const eventsUtil=(el,event,handler,option)=>{
        if(option==='on'){
            el.addEventListener(event,handler)
        } else if (option==='off'){
            el.removeEventListener(event,handler)
        }
    }
    const eventsBatch=(el,Events,handler,option)=>{
        Events.forEach((ev)=>{
            eventsUtil(el,ev,handler,option)
        })
    }
    Array.prototype.remove=function(item){
        if(!this.length) return;
        let index=this.indexOf(item)
        if(index>-1) Array.prototype.splice.call(this,index,1)
    }
    export default{
        props:{
            placeholder:String,
            src:String,
            maxWidth:Number,
            maxHeight:Number,


        },
        data(){
            return{
                paddingBottom:75,
                isLoaded:false
            }
        },
        computed:{
            canvas(){
                return this.$refs.canvas
            },
            width(){
                    return this.maxWidth  || 0
            },
            height(){
                return this.maxHeight  || 0
            }

        },
        methods:{
            handleImgLoad(){
                this.isLoaded=true;
            },
            handlePlaceholderLoad(ev){
                let placeholder=ev.target
                this.paddingBottom=placeholder.height/placeholder.width*100;
                StackBlur.image(this.$refs.placeholder,this.canvas,20)
            },

        },
        created(){
            listeners.push(this)
            eventsBatch(window,EVENTS,lazyLoad,'on')
        },
        beforeDestroy(){
            listeners.remove(this)
            imgCache.remove(this)
            if(listeners.length===0) eventsBatch(window,EVENTS,lazyLoad,'off')
        }

    }
</script>
<style>
    .progressive{
        width:100%;
        margin:0 auto;
        position:relative;
    }

    .progressive-img{
        display: block;
        width:100%;
        height:100%;
        position:absolute;
        top:0;
        left:0;
    }
    .progressive-fill{
        height: 0;
        display:block;
    }

    .fade-enter, .fade-leave-to{
        opacity:0
    }
    .fade-enter-active, .fade-leave-active{
        transition:opacity 0.5s;
    }
</style>
