<template>
    <g> 
        <circle :r=radius :fill="fill" :cx=x :cy=y></circle>
    </g>
</template>
<script>
export default {
    props:{
        x:Number,
        y:Number,
        type:String,
        radius:Number,
        id:Number,
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