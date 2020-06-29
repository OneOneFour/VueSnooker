<template>
  <div id="app" tabindex=-1>
    <h1> Awesome Pool</h1>
    <svg width=700 height=400  >
      <pool-table :balls=balls :x="0" :y="0" :size="700" :ballRadius="ballRadius"/>
    </svg>
    <div> 
      <p> Kinetic Energy : {{kineticEnergy}} </p>
    </div>

  </div>
</template>

<script>
//GLOBAL CONSTANTS
const pocketRadius = 12;
function checkPocket(x,y){
  // Returns the coordinates of the pocket nearest to the ball for checking to see if the ball is in the pocket
  if(x < -50){
    return nearestPocketCheck(x,y,-104, (y>0)? 53: -53);
  }else if(x > 50){
    return nearestPocketCheck(x,y,104, (y>0)? 53: -53);
  }else{
    return nearestPocketCheck(x,y, 0, (y>0)? 59: -59);
  }
}

function nearestPocketCheck(ballX,ballY,pocketX,pocketY){
  return Math.sqrt(Math.pow(pocketX-ballX,2) + Math.pow(pocketY-ballY,2)) < 10 + pocketRadius;
}

function Ball(x,y,vx,vy,type){
  this.x = x;
  this.y = y;
  this.vx = vx;
  this.vy = vy;
  this.type = type;
  this.collided = []; 
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
  },
  destroyed(){
    window.removeEventListener("keydown",this.pause);
  },
  data(){
    return{
      intervalId:null,
      balls:[
        new Ball(-15,0,100,0,"cue"),
        new Ball,47.5,-4.5,0,0,"red"),
        new Ball(47.5,4.5,0,0,"yellow"),
        new Ball()
      ],
      restitutionCoeff:0.92,
      staticFriction: 10,
      ballRadius:4,
      frictionCoeff:0.005 // I looked up this is the actual friction coeff of a pool table!
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
      let frameCount = 0; 
      this.intervalId = setInterval(function(){
        let col =false;
        if(this.balls.filter( x=> !x.stationary()).length == 0){
           clearInterval(this.intervalId);
            this.intervalId = null;
             console.log(frameCount)
        }
        for(let ball of this.balls){
          // Check if intersecting a pocket
          checkPocket(ball.x,ball.y);
          // Check if intersecting another ball
          let colList = this.balls.filter(x=> !(ball.collided.includes(x)))
                                  .filter(x=> x !== ball)
                                  .filter(otherBall=>Math.pow(otherBall.x - ball.x,2) + Math.pow(otherBall.y - ball.y,2) < 4*this.ballRadius*this.ballRadius)
          for(let otherBall of colList){
            let distance = Math.sqrt(Math.pow(otherBall.x - ball.x,2) + Math.pow(otherBall.y - ball.y,2))
            let Vx = (ball.vx + otherBall.vx)/2;
            let Vy = (ball.vy + otherBall.vy)/2;
            let initAngle = Math.atan2(Vy,Vx);
            let b = Math.sqrt(1 - Math.pow((otherBall.x - ball.x)*Vx + Vy*(otherBall.y - ball.y),2)/(distance*distance*(Math.pow(Vx,2) + Math.pow(Vy,2))));
            let impactAngle = Math.asin(b) * Math.sign(otherBall.y - ball.y);
            let u1x = ball.vx - Vx;
            let u2x = otherBall.vx - Vx;
            let u1y = ball.vy - Vy;
            let u2y = otherBall.vy - Vy;
            ball.vx = Vx -u1x*this.restitutionCoeff*Math.cos(impactAngle - initAngle) + u1y*this.restitutionCoeff*Math.sin(impactAngle - initAngle);
            otherBall.vx = Vx -u2x*this.restitutionCoeff*Math.cos(impactAngle - initAngle) + u2y*this.restitutionCoeff*Math.sin(impactAngle - initAngle);
            ball.vy = Vy - u1x*this.restitutionCoeff*Math.sin(impactAngle - initAngle) - u1y*this.restitutionCoeff*Math.cos(impactAngle - initAngle);
            otherBall.vy = Vy - u2x*this.restitutionCoeff*Math.sin(impactAngle - initAngle) - u2y*this.restitutionCoeff*Math.cos(impactAngle - initAngle);
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
          ball.vx -= ((this.frictionCoeff)*ball.vx + this.staticFriction)*Math.cos(angle)*(1/60);
          ball.vy -= ((this.frictionCoeff)*ball.vy + this.staticFriction)*Math.sin(angle)*(1/60); 
          // Update position
          ball.x += ball.vx * (1/60);
          ball.y += ball.vy * (1/60);

          if(ball.stationary()){
            ball.vx = 0;
            ball.vy = 0;
          }
        }
        for(let ball of this.balls) {
          ball.collided = ball.collided
                                  .filter(otherBall=>Math.pow(otherBall.x - ball.x,2) + Math.pow(otherBall.y - ball.y,2) < 4*this.ballRadius*this.ballRadius);
       }
      if(col) console.log(this.balls[0].collided);
      frameCount++;
      }.bind(this),1000/60); // 60FPS
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
</style>
