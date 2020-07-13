<template>
  <div id="app" tabindex=-1>
    <svg :width="pageWidth" :height="pageHeight" @mousemove="mouseMove" @mousedown="enableCue" @mouseup="disableCue">
      <pool-table :balls=balls :x="centerX" :y="centerY" :size="size" :ballRadius="ballRadius" :showCue="intervalId == null && !freePositionCue" />
    </svg>
    <button v-if="freePositionCue" @click="freePositionCue=false">Cue position OK?</button>
  </div>
</template>

<script>
import {cueBus} from './cueBus.js';

//GLOBAL CONSTANTS
const pocketRadius =6;
function checkPocket(x,y){
  // Returns the coordinates of the pocket nearest to the ball for checking to see if the ball is in the pocket
  if(x < -50){
    return nearestPocketCheck(x,y,-105, (y>0)? 54: -54);
  }else if(x > 50){
    return nearestPocketCheck(x,y,105, (y>0)? 54: -54);
  }else{
    return nearestPocketCheck(x,y, 0, (y>0)? 57: -57);
  }
}

function nearestPocketCheck(ballX,ballY,pocketX,pocketY){
  return Math.sqrt(Math.pow(pocketX-ballX,2) + Math.pow(pocketY-ballY,2)) < 3 + pocketRadius;
}

function Ball(id,x,y,vx,vy,type,wiggle=false){
  this.id = id;
  if(wiggle){
    this.x = x + 0.5*(Math.random() - 0.5);
    this.y = y + 0.5*(Math.random() - 0.5);
  }else{
    this.x = x;
    this.y = y;
  }
  this.vx = vx;
  this.vy = vy;
  this.type = type;
  this.collided = []; 
  this.deleteThisFrame = false;
}
Ball.prototype.stationary = function(){
  return Math.abs(this.vx) < 0.22 && Math.abs(this.vy) < 0.22;
}

import PoolTable from "./components/PoolTable.vue"
export default {
  name: 'App',
  components:{
    PoolTable
  },
  mounted(){
    this.physicsSimulate();
    window.addEventListener("keydown",this.pause);
    cueBus.$on("ballStrike",function(velocity,angle){
      this.cue.vx = -Math.sin(angle/180 * Math.PI)*velocity;
      this.cue.vy = -Math.cos(angle/180 * Math.PI)*velocity;
      this.freePositionCue = false; // Just to be sure.
      this.physicsSimulate();
    }.bind(this));

    cueBus.$on('dragCue',function(payload){
      if(this.freePositionCue){
        this.cue.y = payload.y;
      }
    }.bind(this));
  },
  destroyed(){
    window.removeEventListener("keydown",this.pause);
  },
  data(){
    return{
      intervalId:null,
      size:700,
      balls:[
        new Ball(0,-52,0,0,0,"cue"),
        new Ball(16,40,0,0,0,"red",true),
        new Ball(15,48,-4.5,0,0,"red",true),
        new Ball(14,48,4.5,0,0,"yellow",true),
        new Ball(8,56,0,0,0,"black",true),
        new Ball(12,56,-9,0,0,"yellow",true),
        new Ball(11,56,9,0,0,"red",true),
        new Ball(10,64,-13,0,0,"red",true),
        new Ball(9,64,-4.5,0,0,"yellow",true),
        new Ball(7,64,4.5,0,0,"red",true),
        new Ball(6,64,13,0,0,"yellow",true),
        new Ball(5,71.5,18,0,0,"red",true),
        new Ball(4,71.5,9,0,0,"yellow",true),
        new Ball(3,71.5,0,0,0,"yellow",true),
        new Ball(2,71.5,-9,0,0,"red",true),
        new Ball(1,71.5,-18,0,0,"yellow",true)
        
      ],
      restitutionCoeff:0.9, 
      staticFriction: 10,
      ballRadius:3.5,
      freePositionCue:false,
      frictionCoeff:0.01 // I looked up this is the actual friction coeff of a pool table!
    }
  },
  methods:{
    pause(e){
      if(e.code === "Space"){
        if(this.intervalId != null){
          clearInterval(this.intervalId);
          this.intervalId = null;
        }else{
          this.physicsSimulate();
        }
      }
    },

    physicsSimulate(){
      //let col = false;
      this.intervalId = setInterval(function(){
        
        if(this.balls.filter( x=> !x.stationary()).length == 0){
           clearInterval(this.intervalId);
            this.intervalId = null;
            if(this.balls[0].type !== 'cue'){
              //Spawn new cue in the middle 
              this.balls = [new Ball(0,-52,0,0,0,"cue")].concat(this.balls)
              this.freePositionCue = true;
            }
        }
        for(let ball of this.balls){
          // Check if intersecting a pocket
          if(checkPocket(ball.x,ball.y)){
            ball.deleteThisFrame=true;
            
            continue;
          }
          // Check if intersecting another ball
          let colList = this.balls.filter(x=> !(ball.collided.includes(x)))
                                  .filter(x=> x !== ball)
                                  .filter(x=> !x.deleteThisFrame)
                                  .filter(otherBall=>Math.pow(otherBall.x - ball.x,2) + Math.pow(otherBall.y - ball.y,2) < 4*this.ballRadius*this.ballRadius)
          for(let otherBall of colList){
            //Revert 
            // ball.x -= ball.vx*(1/60);
            // ball.y -= ball.vy*(1/60);
            // otherBall.x -= otherBall.vx*(1/60);
            // otherBall.y -= otherBall.vy*(1/60);
            let distance = Math.sqrt(Math.pow(otherBall.x - ball.x,2) + Math.pow(otherBall.y - ball.y,2));  
            let Vx = (ball.vx + otherBall.vx)/2;
            let Vy = (ball.vy + otherBall.vy)/2;
        

            let b = Math.sqrt(1 - Math.pow((otherBall.x - ball.x)*Vx + Vy*(otherBall.y - ball.y),2)/(distance*distance*(Math.pow(Vx,2) + Math.pow(Vy,2))));
            let impactAngle = (isNaN(b))? 0 : Math.asin(b * Math.sign(otherBall.y - ball.y)*Math.sign(otherBall.x -ball.x));
            let u1x = ball.vx - Vx;
            let u2x = otherBall.vx - Vx;
            let u1y = ball.vy - Vy;
            let u2y = otherBall.vy - Vy; 
            //let oblique = Math.atan2(u1y,u1x);

            ball.vx = Vx -u1x*this.restitutionCoeff*Math.cos(impactAngle) + u1y*this.restitutionCoeff*Math.sin(-impactAngle);
            otherBall.vx = Vx -u2x*this.restitutionCoeff*Math.cos(impactAngle) + u2y*this.restitutionCoeff*Math.sin(-impactAngle);
            ball.vy = Vy - u1x*this.restitutionCoeff*Math.sin(impactAngle) - u1y*this.restitutionCoeff*Math.cos(-impactAngle);
            otherBall.vy = Vy - u2x*this.restitutionCoeff*Math.sin(impactAngle) - u2y*this.restitutionCoeff*Math.cos(-impactAngle);

            // Mark collisions as resolved
            ball.collided.push(otherBall);
            otherBall.collided.push(ball);
 
          }
        }
       
        for(let ball of this.balls.filter(x=>!x.stationary())){
          if(Math.abs(ball.x) > 100) { 
            ball.vx = -ball.vx* this.restitutionCoeff;
            ball.vy = ball.vy * this.restitutionCoeff;
            ball.x = Math.sign(ball.x)*100;
          }
          if(Math.abs(ball.y) > 50 ) {
            ball.vy = -ball.vy * this.restitutionCoeff;
            ball.vx = ball.vx * this.restitutionCoeff;
            ball.y = Math.sign(ball.y)*50;
          }
          let angle = Math.atan2(ball.vy,ball.vx);
          if(ball.collided.length == 0){
          ball.vx -= ((this.frictionCoeff)*ball.vx + this.staticFriction)*Math.cos(angle)*(1/60);
          ball.vy -= ((this.frictionCoeff)*ball.vy + this.staticFriction)*Math.sin(angle)*(1/60); 
          }
          // Update position
          ball.x += ball.vx * (1/60);
          ball.y += ball.vy * (1/60);

          if(ball.stationary()){
            ball.vx = 0;
            ball.vy = 0;
          }
        }
        for(let ball of this.balls) {
          ball.collided = ball.collided //Check which collisions are still active
                              .filter(otherBall=>Math.pow(otherBall.x - ball.x,2) + Math.pow(otherBall.y - ball.y,2) < 4*this.ballRadius*this.ballRadius);
        }
        this.balls = this.balls.filter(x=>!x.deleteThisFrame);

      }.bind(this),1000/60); // 60FPS
      
    },
    mouseMove(e){
      cueBus.$emit('mouseMove',{
        cueX:this.balls[0].x,
        cueY:this.balls[0].y,
        mouseOffsetX:e.clientX,
        mouseOffsetY:e.clientY
      });
    },
    enableCue(){
      cueBus.$emit("enableCue");
    },
    disableCue(){
      cueBus.$emit("disableCue");
    }
  },
  computed:{
    kineticEnergy(){
      return Math.round(this.balls.reduce((acc,x) => acc + Math.pow(x.vx,2) + Math.pow(x.vy,2),0));
    },
    momentumX(){
      return Math.round(this.balls.reduce((acc,x) => acc + x.vx,0));
    },
    momentumY(){
      return Math.round(this.balls.reduce((acc,x) => acc + x.vy , 0));
    },
    pageWidth(){
      return window.innerWidth;
    },
    pageHeight(){
      return window.innerHeight;
    },
    centerX(){
      return (this.pageWidth - this.size)/2;
    },
    centerY(){
      return (this.pageHeight - this.size/2)/2;
    },
    cue(){
      return this.balls[0];
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

}
#app > h1{
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
svg {
  position: absolute;
  left:0;
  top:0;
  z-index: -1;
}
</style>
