<template>
    <g> 
        <circle :r=radius :fill="fill" :cx=x :cy=y @mousedown.stop="dragBall" @mouseup.stop="releaseBall" @mousemove.stop="trackBall"></circle>
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
        trackBall({pageY}){
            if(this.draggingBall && this.type === "cue"){
                console.log(pageY);
                cueBus.$emit('dragCue',{
                    y:this.$parent.toBallY(pageY)
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