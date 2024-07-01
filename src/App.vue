<template>
  <div class="action">
    shift 奔跑
    <br />
    w a s d 控制上下左右
  </div>
</template>

<script setup>
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, watch } from "vue";

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);

camera.position.z = 5;

const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setClearColor(0xffffff, 1);

document.body.appendChild(renderer.domElement);

const clock = new THREE.Clock();
let mixer = null;
let globalGltf = ref(null);
new GLTFLoader().load(
  "./worker.glb",
  function (gltf) {
    globalGltf.value = gltf;
    // 正面翻转
    gltf.scene.position.y = -1;
    scene.add(gltf.scene);

    // //  加载骨骼动作动画模型
    mixer = new THREE.AnimationMixer(gltf.scene);
    // mixer.clipAction(gltf.animations[1]).play();
  },
  undefined,
  function (error) {
    console.error(error);
  }
);

const orbitControls = new OrbitControls(camera, renderer.domElement);

const keys = {
  w: false,
  s: false,
  a: false,
  d: false,
  shift: false,
};

const animateType = ref(0);
const onKeyDown = (event) => {
  switch (event.key) {
    case "w":
      keys.w = true;
      break;
    case "W":
      keys.w = true;
      break;
    case "s":
      keys.s = true;
      break;
    case "S":
      keys.s = true;
      break;
    case "a":
      keys.a = true;
      break;
    case "A":
      keys.a = true;
      break;
    case "d":
      keys.d = true;
      break;
    case "D":
      keys.d = true;
      break;
    case "Shift":
      keys.shift = true;
      break;
  }
};
const onKeyUp = (event) => {
  switch (event.key) {
    case "w":
      keys.w = false;
      break;
    case "W":
      keys.w = false;
      break;
    case "s":
      keys.s = false;
      break;
    case "S":
      keys.s = false;
      break;
    case "a":
      keys.a = false;
      break;
    case "A":
      keys.a = false;
      break;
    case "d":
      keys.d = false;
      break;
    case "D":
      keys.d = false;
      break;
    case "Shift":
      keys.shift = false;
      break;
  }
};

window.addEventListener("keydown", onKeyDown);
window.addEventListener("keyup", onKeyUp);

// 监听动画类型改变
watch(animateType, (val) => {
  switch (val) {
    case 0:
      handle1();
      break;
    case 1:
      handle2();
      break;
    case 2:
      handle3();
      break;
  }
});

const handle1 = () => {
  mixer = new THREE.AnimationMixer(globalGltf.value.scene);
  mixer.clipAction(globalGltf.value.animations[animateType.value]).play();
};
const handle2 = () => {
  mixer = new THREE.AnimationMixer(globalGltf.value.scene);
  mixer.clipAction(globalGltf.value.animations[animateType.value]).play();
};
const handle3 = () => {
  mixer = new THREE.AnimationMixer(globalGltf.value.scene);
  mixer.clipAction(globalGltf.value.animations[animateType.value]).play();
};

function animate() {
  requestAnimationFrame(animate);
  let speed = 0.05;
  // 有移动未按住 shift
  if ((keys.w || keys.s || keys.a || keys.d) && !keys.shift) {
    animateType.value = 2;
  }
  // 有移动按住 shift
  if ((keys.w || keys.s || keys.a || keys.d) && keys.shift) {
    animateType.value = 1;
    speed = 0.1;
  }

  // 模型移动
  if (keys.w) {
    globalGltf.value.scene.rotation.y = Math.PI;
    globalGltf.value.scene.position.z -= speed;
  }
  if (keys.s) {
    globalGltf.value.scene.rotation.y = 0;
    globalGltf.value.scene.position.z += speed;
  }
  if (keys.a) {
    globalGltf.value.scene.rotation.y = -Math.PI / 2;
    globalGltf.value.scene.position.x -= speed;
  }
  if (keys.d) {
    globalGltf.value.scene.rotation.y = Math.PI / 2;
    globalGltf.value.scene.position.x += speed;
  }

  // 组合按键
  if (keys.w && keys.a) {
    globalGltf.value.scene.rotation.y = Math.PI * 1.25;
  }
  if (keys.w && keys.d) {
    globalGltf.value.scene.rotation.y = -Math.PI / 0.75;
  }
  if (keys.s && keys.a) {
    globalGltf.value.scene.rotation.y = -Math.PI * 0.25;
  }
  if (keys.s && keys.d) {
    globalGltf.value.scene.rotation.y = Math.PI * 0.25;
  }

  // 没有移动
  if (!keys.w && !keys.s && !keys.a && !keys.d) {
    animateType.value = 0;
  }

  orbitControls.update();
  if (mixer) {
    mixer.update(clock.getDelta());
  }

  // camera.position.add(direction);
  renderer.render(scene, camera);
}

animate();
</script>

<style lang="scss" scoped>
.action {
  display: flex;
  gap: 10px;
  position: absolute;
  top: 10px;
  left: 10px;
}
</style>
