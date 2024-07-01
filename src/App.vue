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

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);

camera.position.z = 5;
camera.position.y = 1;
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setClearColor(0xffffff, 1);
// 开启阴影
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();
scene.background = new THREE.Color(0xa0a0a0);
// 远处起雾气
scene.fog = new THREE.Fog(0xa0a0a0, 10, 50);
const hemiLight = new THREE.HemisphereLight(0xffffff, 0x8d8d8d, 3);
hemiLight.position.set(100, 100, 0);
scene.add(hemiLight);

// 平行光源
const dirLight = new THREE.DirectionalLight(0xffffff, 1);
dirLight.position.set(5, 10, 7.5);
dirLight.castShadow = true;
dirLight.shadow.camera.top = 10;
dirLight.shadow.camera.bottom = -10;
dirLight.shadow.camera.left = -10;
dirLight.shadow.camera.right = 10;
dirLight.shadow.camera.near = 0.1;
dirLight.shadow.camera.far = 40;
scene.add(dirLight);

// 地面
const groundGeometry = new THREE.PlaneGeometry(100, 100);
const groundMaterial = new THREE.MeshLambertMaterial({
  color: 0xffffff,
  depthWrite: false,
});
const ground = new THREE.Mesh(groundGeometry, groundMaterial);
ground.rotation.x = -Math.PI / 2;
ground.position.y = -1;
// 接收阴影
ground.receiveShadow = true;
scene.add(ground);

const clock = new THREE.Clock();
let mixer = null;
let globalGltf = ref(null);
new GLTFLoader().load(
  "./worker.glb",
  function (gltf) {
    const model = gltf.scene;
    globalGltf.value = gltf;
    // 正面翻转
    model.position.y = -1;
    // 开启阴影

    model.traverse(function (object) {
      if (object.isMesh) object.castShadow = true;
    });

    scene.add(model);
    // //  加载骨骼动作动画模型
    mixer = new THREE.AnimationMixer(model);
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

  // 动画更新
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
