//學習2程式碼所在
//兩個除號(//)代表註解
let circleX, circleY, circleR;
let bgCircles = [];
let starCount = 5;
let stars = [];
let explosionCircles = [];
let explosionTriggered = false;
let explosionCenter;
let explosionFrame = 0;
let explosionDelay = 60; // 爆破延遲幀數
let bubbleExplosions = []; // 新增泡泡爆破資料
let explosionInterval = 120; // 每隔多少幀爆破一次
let popSound;
let score = 0; // 新增分數變數
let timer = 60; // 倒數秒數
let startTime;
let gameOver = false;

function preload() {
  popSound = loadSound('balloon-pop.mp3');
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  circleR = 60;
  circleX = width / 2;
  circleY = height - circleR / 2;

  startTime = millis(); // 記錄開始時間

  for(let i=0;i<30;i++){
    bgCircles.push({
      x: random(width),
      y: random(height),
      r: random(30,120),
      alpha: random(50,255),
      color: color(random(255), random(255), random(255)),
      stars: Array.from({length: starCount}, () => ({
        angle: random(TWO_PI),
        dist: 0,
        speed: random(2, 5),
        size: random(8, 16)
      }))
    });
  }
}

function draw() {
  background('#bde0fe');
  noStroke();

  // 計算剩餘秒數
  if (!gameOver) {
    let elapsed = int((millis() - startTime) / 1000);
    let remain = timer - elapsed;
    if (remain <= 0) {
      remain = 0;
      gameOver = true;
    }

    // 左上角文字
    textSize(24);
    let gradColor1 = lerpColor(color('#3399ff'), color('#87ceeb'), 0.3);
    let gradColor2 = lerpColor(color('#3399ff'), color('#87ceeb'), 0.7);

    fill(gradColor1);
    text('414730779', 20, 30);

    fill(gradColor2);
    text('得分：' + score, 20, 60);

    // 顯示倒數計時（橘色）
    fill(255, 140, 0);
    text('倒數：' + remain + ' 秒', 20, 90);

    // 背景圓持續往上飄
    for(let i=0;i<bgCircles.length;i++){
      let c = bgCircles[i];
      fill(c.color, c.alpha);
      ellipse(c.x, c.y, c.r, c.r);

      let squareSize = c.r / 4;
      let offset = c.r / 4;
      let starX = c.x + offset;
      let starY = c.y - offset;
      drawStar(starX, starY, squareSize / 2, squareSize / 4, 5);

      c.y -= 1.2;
      if(c.y < -c.r/2){
        bubbleExplosions.push({
          x: c.x,
          y: 0,
          r: c.r,
          frame: 0,
          circles: Array.from({length: 8}, () => ({
            angle: random(TWO_PI),
            dist: 0,
            speed: random(3, 7),
            size: random(20, 40),
            color: color(random(255), random(255), random(255)),
            alpha: 255
          }))
        });
        c.y = height + c.r/2;
      }
    }

    // 泡泡爆破動畫
    for(let i=bubbleExplosions.length-1; i>=0; i--){
      let be = bubbleExplosions[i];
      for(let j=0;j<be.circles.length;j++){
        let s = be.circles[j];
        let sx = be.x + cos(s.angle) * s.dist;
        let sy = be.y + sin(s.angle) * s.dist;
        fill(s.color, s.alpha);
        ellipse(sx, sy, s.size, s.size);
        s.dist += s.speed;
        s.alpha -= 8;
      }
      be.frame++;
      if(be.frame > 30){
        bubbleExplosions.splice(i, 1);
      }
    }

    if (explosionTriggered) {
      for(let i=0;i<explosionCircles.length;i++){
        let ec = explosionCircles[i];
        let ex = explosionCenter.x + cos(ec.angle) * ec.dist;
        let ey = explosionCenter.y + sin(ec.angle) * ec.dist;
        fill(ec.color, ec.alpha);
        ellipse(ex, ey, ec.r, ec.r);

        ec.dist += ec.speed;
        ec.alpha -= 6;
      }
      explosionFrame++;
      if (explosionFrame > 60) {
        explosionTriggered = false;
      }
    }
  } else {
    // 遊戲結束畫面
    background('#bde0fe');
    textAlign(CENTER, CENTER);
    textSize(40);
    fill('#3399ff');
    text('遊戲結束！', width/2, height/2 - 40);
    fill('#87ceeb');
    text('你的得分：' + score, width/2, height/2 + 20);
    textAlign(LEFT, BASELINE); // 恢復預設
  }
}

// 畫星星的函數
function drawStar(x, y, radius1, radius2, npoints) {
  let angle = TWO_PI / npoints;
  let halfAngle = angle / 2.0;
  fill(255, 255, 0, 200); // 黃色星星
  beginShape();
  for (let a = 0; a < TWO_PI; a += angle) {
    let sx = x + cos(a) * radius1;
    let sy = y + sin(a) * radius1;
    vertex(sx, sy);
    sx = x + cos(a + halfAngle) * radius2;
    sy = y + sin(a + halfAngle) * radius2;
    vertex(sx, sy);
  }
  endShape(CLOSE);
}

function mousePressed() {
  let hit = false;
  for(let i=0; i<bgCircles.length; i++){
    let c = bgCircles[i];
    let d = dist(mouseX, mouseY, c.x, c.y);
    if(d < c.r/2){
      // 點到氣球
      explosionTriggered = true;
      explosionFrame = 0;
      explosionCenter = { x: c.x, y: c.y };
      // 播放音效
      if (popSound) popSound.play();
      // 重新產生爆破圓
      explosionCircles = [];
      for(let j=0;j<20;j++){
        let angle = random(TWO_PI);
        explosionCircles.push({
          x: explosionCenter.x,
          y: explosionCenter.y,
          r: random(30, 80),
          color: color(random(255), random(255), random(255)),
          angle: angle,
          speed: random(4, 8),
          dist: 0,
          alpha: 255
        });
      }
      score++; // 得分加一
      // 讓氣球消失（重設到畫面下方）
      c.y = height + c.r/2;
      hit = true;
      break; // 只爆破一顆
    }
  }
  // 如果沒點到氣球，則扣分
  if (!hit) {
    score--; // 扣分
    if (score < 0) score = 0; // 分數不低於0
  }
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
