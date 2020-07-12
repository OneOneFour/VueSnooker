<template>
    <g >
        <image :width=size :height="height"  xlink:href="../assets/table.svg" :x=x :y=y /> 
        <pool-ball v-for="ball in balls" :x="toTableX(ball.x)" :y="toTableY(ball.y)" :type="ball.type" :key="ball.id" :radius="radius" :id="ball.id"/>
        <pool-cue v-if="!simulating" :ballRadius="radius" />
    </g>
</template>
<script>
const offsetFrac = (19/137);
const totalRatio = 1.7466;
import PoolBall from "./PoolBall.vue";
import PoolCue from "./PoolCue.vue";
export default {
    components:{
        PoolBall,
        PoolCue
    },
    props:{
        simulating:Boolean,
        balls:Array,
        x:Number,
        y:Number,
        size:Number,
        ballRadius:Number
    },
    methods:{
        toTableX(ballX){
            // Helper function to convert 
            return this.x + this.tableHeight*(2/1.967) + (ballX/100)*this.tableHeight*(2/1.967) + this.boundaryWidth;
        },
        toTableY(ballY){
             return this.y + this.tableHeight/2 + (ballY/50)*this.tableHeight/2 + this.boundaryWidth;
        }
    },
    computed:{
        boundaryWidth(){
            return offsetFrac*this.height;
        },
        height(){
            return this.size/totalRatio;
        },
        tableHeight(){
            return this.height - this.boundaryWidth*2;
        },
        radius(){
            return this.ballRadius*this.tableHeight/100
        }
    }
}
</script>
<style>

</style>