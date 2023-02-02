+++ 
date = 2022-07-23T11:42:34+10:00
title = "Boids"
description = "An artificial life simulation"
slug = ""
authors = ["Benjamin Christian"]
tags = ["javascript","html5"]
categories = []
externalLink = ""
series = []
+++

# 🚧 Broken on Mobile 🏗️

&emsp;'Boids' is an artificial life simulation invented by Craig Reynolds. 
It is an example of artificial life and revolves around a set of simple rules that allow for emergent behavior arising from the interaction of the 'boids' in the simulation. 
The basic rules that define the simulation behavior are outlined below:

- Separation: Boids avoid crashing into other boids.
- Coherence: Boids move to centralise themselves in the flock .
- Alignment: Boids attempt to match their velocity to those around them.

Check it out below, written in JavaScript:

<canvas id="game" width="800" height="800"></canvas>

<script>
/*
    Source code for Boids,
    written by Benjamin Christian
*/

//canvas boilerplate
canvas = document.getElementById("game");
ctx = canvas.getContext("2d");

//simulation comstants
paused = 0;

separationFactor = 0.05;
coherenceFactor = 0.005;
alignmentFactor = 0.01;

//color palette
palette = [
    [244, 124, 124], //coral red
    [247, 244, 139], //flavescent yellow
    [161, 222, 147], //apple green
    [112, 161, 215] //baby blue
];

visualRange = 80;
maxVelocity = 5;
minDistance = 20;

turnRate = 1;
margin = 20;

numBoids = 200;
boids = [];

class Boid {
    constructor() {
        this.pos = [0, 0, 0];
        this.vel = [0, 0, 0];
        this.acc=[0,0,0];
        this.col = palette[Math.round(Math.random() * 3)];
        this.randomise();
    }
    //randomise boid position and velocity
    randomise() {
        for (let k = 0; k < 3; k++) {
            this.pos[k] = Math.random() * 800;
            this.vel[k] = Math.random() * maxVelocity;
        }
        this.draw();
    }
    //calculate distance to other boid
    distance(otherBoid) {
        let dist = 0;
        for (let k = 0; k < 3; k++) {
            dist += Math.pow(this.pos[k] - otherBoid.pos[k], 2);
        }
        return Math.sqrt(dist);
    }
    //fly towards center of flock
    cohere() {
        let center = [0, 0, 0];
        let friends = 0;
        for (let otherBoid of boids) {
            if (this.distance(otherBoid) < visualRange) {
                if (this.col == otherBoid.col) {
                    for (let k = 0; k < 3; k++) {

                        center[k] += otherBoid.pos[k];
                    }
                    friends += 1;
                }
            }
        }
        if (friends) {
            for (let k = 0; k < 3; k++) {
                center[k] = center[k] / friends;
                this.vel[k] += (center[k] - this.pos[k]) * coherenceFactor;
            }
        }
    }
    //match velocity with flock
    align() {
        let velocity = [0, 0, 0];
        let friends = 0;
        for (let otherBoid of boids) {
            if (this.distance(otherBoid) < visualRange) {
                if (this.col == otherBoid.col) {
                    for (let k = 0; k < 3; k++) {
                        velocity[k] += otherBoid.vel[k];
                    }
                    friends += 1;
                }
            }
        }
        if (friends) {
            for (let k = 0; k < 3; k++) {
                velocity[k] = velocity[k] / friends;
                this.vel[k] += (velocity[k] - this.vel[k]) * alignmentFactor;
            }
        }
    }
    //avoid collisions with flock mates
    separate() {
        let move = [0, 0, 0];
        for (let otherBoid of boids) {
            if (otherBoid != this) {
                if (this.distance(otherBoid) < minDistance) {
                    for (let k = 0; k < 3; k++) {
                        move[k] += this.pos[k] - otherBoid.pos[k];
                    }
                }
            }
        }
        for (let k = 0; k < 3; k++) {
            this.vel[k] += move[k] * separationFactor;
        }
    }
    //bound boid to canvas
    bound() {
        for (let k = 0; k < 3; k++) {
            if (this.pos[k] < margin) {
                this.vel[k] += turnRate;
            }
            else if (this.pos[k] > (canvas.width - margin)) {
                this.vel[k] -= turnRate;
            }
        }
    }
    //limit boid velocity
    limit() {
        let velocity = 0;
        for (let k = 0; k < 3; k++) {
            velocity += Math.pow(this.vel[k], 2);
        }
        velocity = Math.sqrt(velocity);
        if (velocity > maxVelocity) {
            for (let k = 0; k < 3; k++) {
                this.vel[k] = (this.vel[k] / velocity) * maxVelocity;
            }
        }
    }
    //move boid position
    move() {
        for (let k = 0; k < 3; k++) {
            this.pos[k] += this.vel[k];
        }
    }
    //call boid methods
    update() {
        this.separate();
        this.cohere();
        this.align();

        this.bound();
        this.limit();

        this.move();
    }
    //draw boid
    draw() {
        ctx.fillStyle = `rgb(${this.col[0]},${this.col[1]},${this.col[2]})`;
        ctx.beginPath();
        ctx.arc(this.pos[0], this.pos[1], 1 + (this.pos[2] / 200), 0, 2 * Math.PI);
        ctx.fill();
    }
}

for (let i = 0; i < numBoids; i++) {
    boids.push(new Boid);
}

addEventListener('keydown', (event) => {
    if (event.key == 'r') {
        for (let i = 0; i < numBoids; i++) {
            boids[i].randomise();
        }
    }
    if (event.key == 'p') {
        paused ^= 1;
    }
})

function animationLoop() {
    requestAnimationFrame(animationLoop);

    now = Date.now();
    if ((now - then) >= interval) {
        then = now;
        if (!paused) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            for (let boid of boids) {
                boid.update();
                boid.draw();
            }
        }
    }
}

//setup animation function
function startLoop(fps) {
    interval = 1000 / fps;
    then = Date.now();
    requestAnimationFrame(animationLoop);
}
startLoop(60);
</script>

&emsp;Let us look at some of the code that allows this to happen 🧐.
Firstly, each boid is an object and all interactions are method driven.
The constructor firstly initialises the position, velocity, acceleration and colour for each boid, which is pulled from a defined palette.

```
constructor() {
    this.pos = [0, 0, 0];
    this.vel = [0, 0, 0];
    this.acc=[0,0,0];
    this.col = palette[Math.round(Math.random() * 3)];
    this.randomise();
}
```

&emsp;The coherence behaviour is defined by the following method:

```
//fly towards center of flock
cohere() {
    let center = [0, 0, 0];
    let friends = 0;
    for (let otherBoid of boids) {
        if (this.distance(otherBoid) < visualRange) {
            if (this.col == otherBoid.col) {
                for (let k = 0; k < 3; k++) {

                    center[k] += otherBoid.pos[k];
                }
                friends += 1;
            }
        }
    }
    if (friends) {
        for (let k = 0; k < 3; k++) {
            center[k] = center[k] / friends;
            this.vel[k] += (center[k] - this.pos[k]) * coherenceFactor;
        }
    }
}
```

&emsp;Aligment...

```
//match velocity with flock
align() {
    let velocity = [0, 0, 0];
    let friends = 0;
    for (let otherBoid of boids) {
        if (this.distance(otherBoid) < visualRange) {
            if (this.col == otherBoid.col) {
                for (let k = 0; k < 3; k++) {
                    velocity[k] += otherBoid.vel[k];
                }
                friends += 1;
            }
        }
    }
    if (friends) {
        for (let k = 0; k < 3; k++) {
            velocity[k] = velocity[k] / friends;
            this.vel[k] += (velocity[k] - this.vel[k]) * alignmentFactor;
        }
    }
}
```

&emsp;And finally separation...

```
//avoid collisions with flock mates
separate() {
    let move = [0, 0, 0];
    for (let otherBoid of boids) {
        if (otherBoid != this) {
            if (this.distance(otherBoid) < minDistance) {
                for (let k = 0; k < 3; k++) {
                    move[k] += this.pos[k] - otherBoid.pos[k];
                }
            }
        }
    }
    for (let k = 0; k < 3; k++) {
        this.vel[k] += move[k] * separationFactor;
    }
}
```

&emsp;These three rules, implemented as separate functions operate on every boid in the simulation. 
Importantly, they are not equally weighted and each will influence the simulation differently, allowing specific simulations to be 'tuned', similar to a PID controller. Additional rules can be implemented to the simulation to provide more complex operation such as obstacle avoidance, however I will not explore these concepts here.
You can randomise the above simulation using 'r' and pause using 'p'. 

---
Some additional additional reading material(s):
- [Source Code](https://github.com/bfkxtian/boids/blob/main/boids.js)
- [Wikipedia](https://en.wikipedia.org/wiki/Boids)
- [Ben Eater](https://eater.net/boids)