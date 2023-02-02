+++ 
date = 2023-01-03T11:42:34+10:00
title = "The Game of Life"
description = "A simple cellular automata"
slug = ""
authors = ["Benjamin Christian"]
tags = ["javascript","html5"]
categories = []
externalLink = ""
series = []
+++

# 🚧 Broken on Mobile 🏗️

&emsp;'Life' is a cellular automata invented by the late John Conway.
It is a discrete simulation consisting of a (theoretically) infinite two-dimensional grid of cells.
Each of these cells can be either alive or dead and has eight neighbours which affects its next state.
There are three rules that define life, outlined as:
1. Any living cell with two or three live neighbours will continue living. ❤️
2. Any dead cell with three live neighbours will be brought to life. 🌱
3. All other cells will die, dead cells will remain dead. ☠️

I have written an implementation of the game of life in JavaScript below:

<canvas id="game" width="800" height="800"></canvas>
<script>
/*
    Source code for Game of Life,
    written by Benjamin Christian
*/

//canvas boilerplate
canvas = document.getElementById("game");
ctx = canvas.getContext("2d");

paused = 0;

//board class
class Board {
    constructor() {
        this.cell = 5;
        this.grid = 160;
        this.board = new Array(this.grid);
        this.friends = new Array(this.grid);
        for (let x = 0; x < this.grid; x++) {
            this.board[x] = new Array(this.grid);
            this.friends[x] = new Array(this.grid);
        }
    }
    //setter method
    set(x, y, state) {
        x = Math.abs(x) % this.grid; y = Math.abs(y) % this.grid;
        this.board[x][y] = state;
    }
    //getter method
    alive(x, y) {
        x = Math.abs(x) % this.grid; y = Math.abs(y) % this.grid;
        return this.board[x][y];
    }
    //randomise board state
    randomise() {
        for (let x = 0; x < this.grid; x++) {
            for (let y = 0; y < this.grid; y++) {
                if (Math.random() <= 0.1) {
                    this.set(x, y, 1);
                }
                else {
                    this.set(x, y, 0);
                }
            }
        }
        this.draw();
    }
    //clear board state
    clear() {
        for (let x = 0; x < this.grid; x++) {
            for (let y = 0; y < this.grid; y++) {
                this.set(x, y, 0);
                this.friends[x][y] = 0;
            }
        }
        this.draw();
    }
    update() {
        //count neighbours
        for (let x = 0; x < this.grid; x++) {
            for (let y = 0; y < this.grid; y++) {
                this.friends[x][y] = 0;
                for (let i = -1; i < 2; i++) {
                    for (let j = -1; j < 2; j++) {
                        //ignore self-comparison case
                        if ((i == 0) && (j == 0)) {
                            continue;
                        }
                        this.friends[x][y] += this.alive(x + i, y + j);
                    }
                }
            }
        }
        //implement rules
        for (let x = 0; x < this.grid; x++) {
            for (let y = 0; y < this.grid; y++) {
                if (this.friends[x][y] < 2) {
                    this.set(x, y, 0);
                }
                else if (this.friends[x][y] > 3) {
                    this.set(x, y, 0);
                }
                else if (this.friends[x][y] === 3) {
                    this.set(x, y, 1);
                }
            }
        }
        this.draw();
    }
    //draw board to display
    draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.beginPath();
        ctx.fillStyle = "rgb(112,161,215)";
        for (let x = 0; x < this.grid; x++) {
            for (let y = 0; y < this.grid; y++) {
                if (this.board[x][y]) {
                    ctx.fillRect((x - 1) * this.cell, (y - 1) * this.cell, this.cell, this.cell);
                }
            }
        }
    }
}
//create board object
board = new Board();

//keyboard handler
addEventListener('keydown', (event) => {
    if (event.key == 'r') {
        board.randomise();
    }
    if (event.key == 'p') {
        paused ^= 1;
    }
    if (event.key == 'c') {
        board.clear();
    }
})

//mouse handler
addEventListener('click', (event) => {
    let rect = canvas.getBoundingClientRect();
    mouseX = Math.round(event.clientX - rect.left);
    mouseY = Math.round(event.clientY - rect.top);

    gridX = Math.round(mouseX / board.cell);
    gridY = Math.round(mouseY / board.cell);

    if (paused) {
        board.set(gridX, gridY, 1);
    }
    board.draw();

    console.log(gridY, gridX);
})

//animation loop
function animationLoop() {
    requestAnimationFrame(animationLoop);

    now = Date.now();
    if ((now - then) >= interval) {
        then = now;
        if (!paused) {
            board.update();
        }
    }
}

//start looping
function startLoop(fps) {
    board.randomise();
    interval = 1000 / fps;
    then = Date.now();
    requestAnimationFrame(animationLoop);
}
startLoop(60);
</script>

&emsp;To understand what's going on here, we can look at the source code (link down the bottom! 😉).
The constructor method creats a grid of cells, then randomises the state.

```
constructor() {
    this.cell = 5;
    this.grid = 160;
    this.board = new Array(this.grid);
    this.friends = new Array(this.grid);
    for (let x = 0; x < this.grid; x++) {
        this.board[x] = new Array(this.grid);
        this.friends[x] = new Array(this.grid);
    }
}
```

&emsp;And then the next board state is calculated entirely before drawing the next frame. 
This is important as cell states need to be considered holistically to ensure that the next board state can be effectively calculated.

```
update() {
    //count neighbours
    for (let x = 0; x < this.grid; x++) {
        for (let y = 0; y < this.grid; y++) {
            this.friends[x][y] = 0;
            for (let i = -1; i < 2; i++) {
                for (let j = -1; j < 2; j++) {
                    //ignore self-comparison case
                    if ((i == 0) && (j == 0)) {
                        continue;
                    }
                    this.friends[x][y] += this.alive(x + i, y + j);
                }
            }
        }
    }
    //implement rules
    for (let x = 0; x < this.grid; x++) {
        for (let y = 0; y < this.grid; y++) {
            if (this.friends[x][y] < 2) {
                this.set(x, y, 0);
            }
            else if (this.friends[x][y] > 3) {
                this.set(x, y, 0);
            }
            else if (this.friends[x][y] === 3) {
                this.set(x, y, 1);
            }
        }
    }
    this.draw();
}
```

&emsp;Perhaps unsurprisingly, there are actually two types of stable life, those that remain in place (static on the board) and those that move (these are a bit more exciting to observe in action)...
See if you can spot both types 🧐.

---
Some additional reading material(s):
- [Source Code](https://github.com/bfkxtian/game-of-life/blob/main/life.js)
- [Wikipedia](https://en.wikipedia.org/wiki/Conway's_Game_of_Life)