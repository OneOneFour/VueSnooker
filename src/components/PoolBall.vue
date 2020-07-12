<template>
    <g> 
        <circle :r=radius :fill="fill" :cx=x :cy=y @mousedown="dragBall" @mouseup="releaseBall" @mousemove="trackBall"></circle>
    </g>
</template>
<script>
import {cueBus} from '../cueBus.js';
export default {
    props:{
        x:Number,
        y:Number,
        type:String,
        radius:Number,
        id:Number,
    },
    data(){
        return{
            draggingBall:false
        }
    },
    methods:{
        dragBall(){
            this.draggingBall = true;
        },
        releaseBall(){
            this.draggingBall = false;
        },
        trackBall({clientX,clientY}){
            if(this.draggingBall && this.type === "cue"){
                cueBus.$emit('dragCue',{
                    x:this.$parent.toTableX(clientX),
                    y:this.$parent.toTableY(clientY)
                });
            }
        }
    },
    computed:{
        fill(){
            switch(this.type){
                case "cue":
                    return "#f0ead6"
                case "red":
                    return "#c0392b"
                case "yellow":
                    return "#f1c40f"
                case "black":
                    return "#2d3436"
                default:
                    return "#000000"
            }
        }
    }
}
</script>