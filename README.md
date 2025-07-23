# wpro-antiviruss
wpro antivirus
// Brick Ball 3D - JavaScript Web Game (Version 2025)
// Unique 4K WebGL-powered version using Three.js + Levels + Score

import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

// Setup renderer
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// Setup scene and camera
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000000);

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.z = 10;

// Controls
const controls = new OrbitControls(camera, renderer.domElement);

// Light
const light = new THREE.PointLight(0xffffff, 1);
light.position.set(10, 10, 10);
scene.add(light);

// Paddle
const paddleGeometry = new THREE.BoxGeometry(2, 0.5, 0.5);
const paddleMaterial = new THREE.MeshStandardMaterial({ color: 0xffffff });
const paddle = new THREE.Mesh(paddleGeometry, paddleMaterial);
paddle.position.y = -4;
scene.add(paddle);

// Ball
const ballGeometry = new THREE.SphereGeometry(0.3, 32, 32);
const ballMaterial = new THREE.MeshStandardMaterial({ color: 0xffff00 });
const ball = new THREE.Mesh(ballGeometry, ballMaterial);
ball.position.set(0, -3.5, 0);
scene.add(ball);

let ballVelocity = new THREE.Vector3(0.1, 0.1, 0);

// Score and Level Display
let score = 0;
let level = 1;
const scoreDiv = document.createElement('div');
scoreDiv.style.position = 'absolute';
scoreDiv.style.top = '20px';
scoreDiv.style.left = '20px';
scoreDiv.style.color = 'white';
scoreDiv.style.fontSize = '20px';
document.body.appendChild(scoreDiv);

// Bricks
let bricks = [];
const brickSpacing = 0.5;
const brickSize = { width: 2, height: 0.6, depth: 0.3 };

function createBricks(rows, cols) {
  // Remove old bricks
  bricks.forEach(b => scene.remove(b));
  bricks = [];

  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      const brickGeometry = new THREE.BoxGeometry(
        brickSize.width,
        brickSize.height,
        brickSize.depth
      );
      const brickMaterial = new THREE.MeshStandardMaterial({
        color: new THREE.Color(`hsl(${(row * 40 + col * 10) % 360}, 100%, 50%)`),
      });
      const brick = new THREE.Mesh(brickGeometry, brickMaterial);
      brick.position.x =
        col * (brickSize.width + brickSpacing) -
        ((cols - 1) * (brickSize.width + brickSpacing)) / 2;
      brick.position.y = 3 - row * (brickSize.height + 0.3);
      scene.add(brick);
      bricks.push(brick);
    }
  }
}

createBricks(5, 7);

function updateScore() {
  scoreDiv.innerHTML = `🎯 Score: ${score} | 🚀 Level: ${level}`;
}

// Game loop
function animate() {
  requestAnimationFrame(animate);

  // Ball movement
  ball.position.add(ballVelocity);

  // Wall collision
  if (ball.position.x <= -8 || ball.position.x >= 8) {
    ballVelocity.x *= -1;
  }
  if (ball.position.y >= 6) {
    ballVelocity.y *= -1;
  }

  // Paddle collision
  if (
    ball.position.y <= paddle.position.y + 0.4 &&
    ball.position.x > paddle.position.x - 1.1 &&
    ball.position.x < paddle.position.x + 1.1
  ) {
    ballVelocity.y *= -1;
  }

  // Brick collision
  for (let i = 0; i < bricks.length; i++) {
    const b = bricks[i];
    if (!b.visible) continue;

    const dist = ball.position.distanceTo(b.position);
    if (dist < 1.1) {
      b.visible = false;
      score += 10;
      updateScore();
      ballVelocity.y *= -1;
      break;
    }
  }

  // Check if all bricks destroyed
  if (bricks.every(b => !b.visible)) {
    level++;
    ballVelocity.multiplyScalar(1.1);
    createBricks(5 + level, 7);
  }

  renderer.render(scene, camera);
}

// Paddle control
window.addEventListener('mousemove', (event) => {
  const x = (event.clientX / window.innerWidth) * 2 - 1;
  paddle.position.x = x * 8;
});

updateScore();
animate();
