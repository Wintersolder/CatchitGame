# /** @type {HTMLCanvasElement} */
const canvas = document.getElementById("game-canvas");
const ctx = canvas.getContext("2d");
canvas.width = 800;
canvas.height = 600;
canvas.width2 = 0;

let goodBlockRatio = 0.25;
let blockSpawnRate = 500;
const BLOCK_SIZE = 32;

let player = {
  x: 350,
  x2: 740,
  y: canvas.height - BLOCK_SIZE * 3,
  width: BLOCK_SIZE * 2,
  height: BLOCK_SIZE / 2,

  isMovingLeft: false,
  isMovingRight: false,
  speed: 10,

  update: function () {
    if (this.isMovingLeft) this.x -= this.speed;
    if (this.isMovingRight) this.x += this.speed;

    if (this.isMovingLeft && this.x <= 0) this.x = 740;
    if (this.isMovingRight && this.x >= 740) this.x = 0;
  },
  render: function () {
    ctx.save();
    ctx.fillStyle = "black";
    ctx.fillRect(this.x, this.y, this.width, this.height);
    ctx.restore();
  },
};

let scoreBoard = {
  goodTally: 0,
  badTally: 0,
  scoreBlock: function (block) {
    if (block.isGoodBlock) {
      this.goodTally++;
    } else {
      this.badTally++;
    }
  },
};

window.addEventListener("keydown", (e) => {
  if (e.key === "ArrowLeft" || e.key === "a" || e.key === "A")
    player.isMovingLeft = true;
  if (e.key === "ArrowRight" || e.key === "d" || e.key === "D")
    player.isMovingRight = true;
});

window.addEventListener("keyup", (e) => {
  if (e.key === "ArrowLeft" || e.key === "a" || e.key === "A")
    player.isMovingLeft = false;
  if (e.key === "ArrowRight" || e.key === "d" || e.key === "D")
    player.isMovingRight = false;
});

class Block {
  constructor() {
    this.width = BLOCK_SIZE;
    this.height = this.width;
    this.x = Math.random() * (canvas.width - this.width);
    this.y = 0 - this.height; // off screen to start
    this.speed = Math.random() * 10 + 1;
    this.isGoodBlock = Math.random() <= goodBlockRatio;
    this.isOffscreen = false;
    this.isCaught = false;
    this.isScored = false;

    this.isFading = false;
    this.opacity = 1;
    this.color = ctx.fillStyle = this.isGoodBlock ? 120 : 0;
  }

  update() {
    this.y += this.speed;
    this.isOffscreen = this.y >= canvas.height;
    this.checkForCatch();
    if (this.isFading) this.opacity -= 0.1;
  }

  render() {
    ctx.save();

    ctx.fillStyle = `hsla(${this.color}, 100%, 50%, ${this.opacity})`;
    ctx.fillRect(this.x, this.y, this.width, this.height);

    ctx.restore();
  }

  checkForCatch() {
    let bottom = this.y + this.height;

    // if I am above the catch block, return
    if (bottom < player.y) return;
    if (this.isFading || this.isOffscreen || this.isCaught) return;

    let rhs = this.x + this.width;
    if (rhs < player.x || this.x > player.x + player.width) {
      this.isFading = true;
      return;
    }

    scoreBoard.scoreBlock(this);
    this.isCaught = true;
  }
}

// let myBlock = new Block();
// console.log(myBlock);

let blocks = [new Block()];
let currentTime = 0;
let timeSinceLastBlock = 0;

function gameLoop(timestamp) {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  let changeInTime = timestamp - currentTime;
  currentTime = timestamp;

  timeSinceLastBlock += changeInTime;
  if (timeSinceLastBlock >= blockSpawnRate) {
    timeSinceLastBlock = 0;
    blocks.push(new Block());
  }

  blocks.forEach((block) => {
    block.update();
    block.render();
  });

  blocks = blocks.filter((b) => !b.isOffscreen && !b.isCaught);
  //console.log(blocks);

  player.update();
  player.render();

  requestAnimationFrame(gameLoop);
}

requestAnimationFrame(gameLoop);
