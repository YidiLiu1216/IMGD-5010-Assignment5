# IMGD-5010-Assignment5
## Link:
[The source Code link](https://editor.p5js.org/YidiLiu1216/full/KiuPxN4q4)
## What I attempt to create:
I am planning to implement Boids flocking algorithm in this assignment.
1. I randomize the start position and color of 200 agents.
2. For each agent, I implement the Separation, Alignment and Cohesion rule.
3. Call to recaculate the acceleration of each agent each frame to update their position in the canvas.
4. Add an obstacle each time mouse clicked, the flock will try to avoid the obstacle

The result is successed , the triangles behavior really like flock, and they success to avoid obstacle. 
## Full Code

```
agents=[];
agentnum=200;
minspeed=1;
maxspeed=3;
maxforce=0.08;
shapesize=10;

voidrange=25;
voidmodifier=1.1;
alignrange=65;
alignmodifier=0.8;
cohesionrange=50;
cohesionmodifier=1.0;

mousemodifier=1.0;
obstacles=[];
obstaclerange=5;
obstaclemodifier=2.0;

function setup() {
  createCanvas(400, 400);
  
  colorMode(HSB);
  angleMode(RADIANS);
  for(let i = 0; i < agentnum ; i++){
    agents.push( new Agent() );
  }
}

function draw() {
  background(210,220,10);
  for(let i=0;i<agentnum;i++){
    agents[i].update();
  }
  drawobstacle();
}
function drawobstacle(){
  noStroke();
  fill(255);
  for(let i=0;i<obstacles.length;i++){
    circle(obstacles[i].pos.x,obstacles[i].pos.y,obstacles[i].r);
  }
  
}
function mousePressed() {
  obstacles.push({ pos: createVector(mouseX, mouseY), r: 30 });
}

class Agent{
  constructor(){
    this.pos=createVector(random(width),random(height));
    this.color=color(180,random(0,255),random(0,255));
    this.dir=random(TWO_PI);
    this.speed=random(minspeed,maxspeed);
    this.vel= p5.Vector.random2D().mult(random(maxspeed));
    this.size = shapesize;
    this.acc=createVector(0,0);
  }
  update(){
    this.seprule();
    this.cohesionrule();
    this.alignrule();
    //this.applymouse();
    this.avoidobstacle();
    this.calculateSpeed();
    this.drawTriangleBasedonDir();
    
  }
  calculateSpeed(){
    //this.speed=random(minspeed,maxspeed);
    this.vel.add(this.acc);
    this.vel.limit(maxspeed);

    
    //let newx= this.pos.x+cos(this.dir)*this.speed;
    //let newy=this.pos.y+sin(this.dir)*this.speed;
    let newx=this.pos.x+this.vel.x;
    let newy= this.pos.y+this.vel.y;
    //warp the screen
    this.acc.mult(0);
    if(newx>width){newx-=width; }
    if(newx<0){newx+=width;}
    if(newy>height){newy-=height;}
    if(newy<0){newy+=height;}
    this.pos.x=newx;
    this.pos.y=newy;
  }
  
  drawTriangleBasedonDir(){
    push();
    translate(this.pos.x, this.pos.y);
    rotate(this.vel.heading());
    noStroke();
    fill(this.color);
    triangle( this.size, 0,-this.size * 0.6, -this.size * 0.5,-this.size * 0.6,  this.size * 0.5);
    pop();
  }

  seprule(){
    let sumdir = createVector(0, 0);
    let count=0;
    for(let i=0;i<agentnum;i++){
      let dis = dist(agents[i].pos.x,agents[i].pos.y,this.pos.x,this.pos.y);
      if(dis<voidrange&&dis!=0){
        let difvec=p5.Vector.sub(this.pos,agents[i].pos);
        difvec.normalize();
        difvec.div(dis);
        sumdir.add(difvec);
        count+=1;
      }
    }
    if(count>0){sumdir.div(count);}
    sumdir.setMag(maxspeed);
    sumdir.sub(this.vel);
    sumdir.limit(maxforce);
    this.acc.add(sumdir.mult(voidmodifier));
    //let turn = sumdir.heading()- this.dir;
    //this.dir+=this.angleDiff(turn, this.dir) * voidmodifier;
    
    
  }
  
  cohesionrule(){
    let sumpos = createVector(0, 0);
    let count=0;
    for(let i=0;i<agentnum;i++){
      let dis = dist(agents[i].pos.x,agents[i].pos.y,this.pos.x,this.pos.y);
      if(dis<cohesionrange&&dis!=0){
          sumpos.add(agents[i].pos);
          count+=1;
      }
      
    }
    if(count>0){sumpos.div(count);}
    let cohesiondir=p5.Vector.sub(sumpos,this.pos);
    cohesiondir.setMag(maxspeed);
    let steer = p5.Vector.sub(cohesiondir, this.vel);
    steer.limit(maxforce);
    this.acc.add(steer.mult(cohesionmodifier));
    //let turn=cohesiondir.heading();
    //this.dir+=this.angleDiff(turn, this.dir) * cohesionmodifier;
  }
  
  alignrule(){
    let sumdir = createVector(0, 0);
    let count=0;
    for(let i=0;i<agentnum;i++){
      let dis = dist(agents[i].pos.x,agents[i].pos.y,this.pos.x,this.pos.y);
      if(dis<alignrange&&dis!=0){
         sumdir.add(agents[i].vel);
         count+=1;
      }
  }
    if(count>0){sumdir.div(count);}
    sumdir.setMag(maxspeed);
    let steer = p5.Vector.sub(sumdir, this.vel);
    steer.limit(maxforce);
    this.acc.add(steer.mult(alignmodifier));
    //let turn = sumdir.heading();
    //this.dir+=this.angleDiff(turn, this.dir)*alignmodifier;
  } 
  applymouse(){
    let mousepos=createVector(mouseX,mouseY);
    let mousedir=p5.Vector.sub(mousepos, this.pos);
    mousedir.setMag(maxspeed);
    let steer = p5.Vector.sub(mousedir, this.vel);
    steer.limit(maxforce);
    this.acc.add(steer.mult(mousemodifier));
  }
  angleDiff(target, current) {
  return atan2(sin(target - current), cos(target - current));
  }
  avoidobstacle(){
     let sumdir = createVector(0, 0);
    let count=0;
    for(let i=0;i<obstacles.length;i++){
      let dis = dist(obstacles[i].pos.x,obstacles[i].pos.y,this.pos.x,this.pos.y);
      if(dis<obstacles[i].r+obstaclerange&&dis!=0){
        let difvec=p5.Vector.sub(this.pos,obstacles[i].pos);
        difvec.normalize();
        difvec.div(dis);
        sumdir.add(difvec);
        count+=1;
      }
    }
    if(count>0){sumdir.div(count);}
    if (count === 0) return;
    sumdir.setMag(maxspeed);
    sumdir.sub(this.vel);
    sumdir.limit(maxforce);
    this.acc.add(sumdir.mult(obstaclemodifier));
  
  }

}
```
