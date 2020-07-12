<template>
    <image class="cue" :opacity="(clicked)? 1:0.5" xlink:href="../assets/cue.svg" :x="cueX-ballRadius*2.5" :y="cueY + yOffset + extraOffset" :transform="cueStyle" :style="getOrigin"/>
</template>
<script>
import {cueBus} from '../cueBus.js';

export default {
    props:{
        ballRadius:Number
    },
    data(){
        return{
            clicked:false,
            cueX:0,
            cueY:0,
            mouseOffsetX:0,
            mouseOffsetY:0,
            rotationLock:null,
            yOffset:1.5*this.ballRadius,
            velocity:0
        }
    },
    created(){
        let vm = this;
        // TODO : These lines of code are a crime against humanity;
        this.cueX = this.$parent.toTableX(this.$parent.$parent.balls[0].x);
        this.cueY = this.$parent.toTableY(this.$parent.$parent.balls[0].y);
        cueBus.$on('mouseMove',function(payload){
            // TODO: Does a shorthand exist for this in JavaScript?
            vm.mouseOffsetX = payload.mouseOffsetX - vm.cueX;
            vm.mouseOffsetY = payload.mouseOffsetY - vm.cueY;
        });
        cueBus.$on('enableCue',function(){
            vm.rotationLock = {
                rot:vm.rotation,
                x: vm.mouseOffsetX,
                y:vm.mouseOffsetY
            }
            vm.clicked=true;
        });
        cueBus.$on('disableCue',function(){
            vm.clicked=false;
        });
    },
    computed:{
        cueStyle(){
                return "rotate("+ (-1*this.rotation)+")"

        },
        getOrigin(){
            return {
                "transform-origin":`${this.cueX}px ${this.cueY}px`
            }
        },
        rotation(){         
            return (this.clicked)? this.rotationLock.rot :Math.atan2(this.mouseOffsetX,this.mouseOffsetY)*(180/Math.PI);
        },
        basisVector(){
            if(this.clicked){
                let magnitude = Math.sqrt( Math.pow(this.rotationLock.x,2) + Math.pow(this.rotationLock.y,2));
                return [(this.rotationLock.x)/magnitude,(this.rotationLock.y)/magnitude];
            }else{
                return 0;
            }
        },
        extraOffset(){
            if(this.clicked){
                return this.basisVector[0]*(this.mouseOffsetX-this.rotationLock.x) + this.basisVector[1]*(this.mouseOffsetY - this.rotationLock.y);
            }else{
                return 0;
            }
        }
    },
    watch:{
        extraOffset(newValue,oldValue){
            this.velocity = oldValue - newValue; // Needs to be scaled obvs
            if(newValue + this.yOffset < this.ballRadius){
                cueBus.$emit("ballStrike",this.velocity,this.rotationLock.rot);
            }
        }
    }


}
</script>
<style>

</style>