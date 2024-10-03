<script setup>

import { onMounted } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls';
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js';
import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader.js';
import { Reflector } from 'three/addons/objects/Reflector.js';
import EventBus from '../EventBus';

// переменные модели двери
const modelUrl = new URL('doorObj/woodenDoor.obj', import.meta.url).href;
const materialUrl = new URL('doorObj/woodenDoor.mtl', import.meta.url).href;

// переменные размера окна
let wrapper3Dw = window.innerWidth;
let wrapper3Dh = window.innerHeight;

// переменные параметров двери
let door;
let offset = 0;
let delta = 0.001;

// создаём сцену
const scene = new THREE.Scene();
scene.background = new THREE.Color('rgb(0,0,0)');
const renderer = new THREE.WebGLRenderer({antialias: true, powerPreference: 'high-performance'});
renderer.setSize(wrapper3Dw,wrapper3Dh);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap; 
renderer.shadowMapEnabled = true;
renderer.shadowMapSoft = true;
renderer.shadowMapType = THREE.PCFSoftShadowMap;

// создаём камеру
const camera = new THREE.PerspectiveCamera(75,wrapper3Dw/wrapper3Dw,0.1,100);
const controls = new OrbitControls(camera,renderer.domElement);
camera.position.set(-2, 0, 15);
camera.rotation.set(0, -0.3, 0);
camera.aspect = wrapper3Dw / wrapper3Dh;
camera.updateProjectionMatrix();
renderer.setPixelRatio(window.devicePixelRatio); 

// создаём источник освещения
const pointColor = 0xffffff;
const pointIntensity = 1200;
const pointLight = new THREE.PointLight(pointColor,pointIntensity);
pointLight.position.set(0,5,10);
pointLight.rotation.x = -Math.PI*2.1;
pointLight.castShadow = true;
pointLight.shadowCameraFov = 60;
pointLight.shadow.mapSize.x = 1024;
pointLight.shadow.mapSize.y = 1024;
scene.add(pointLight);

// создаём дверь
const objLoader = new OBJLoader();
const mtlLoader = new MTLLoader();
mtlLoader.load(materialUrl, (mtl) => {
  objLoader.setMaterials(mtl);
  objLoader.load(modelUrl, (door) => {
    door.scale.set(0.05,0.05,0.05);
    door.rotation.x = Math.PI/2;
    door.rotation.y = Math.PI;
    door.position.y = -5;
    door.name = "door";
    scene.add(door);
  });
});

// создаём зеркальный пол:

// создаём отражение
const geometry = new THREE.PlaneGeometry(100, 100, 64, 64);
let groundMirror = new Reflector(geometry, {
  clipBias: 0.03,
  textureWidth: wrapper3Dw * window.devicePixelRatio,
  textureHeight: wrapper3Dh * window.devicePixelRatio,
  color: 0xcccccc
});
groundMirror.position.y = -5;
groundMirror.rotateX(-Math.PI/2);
scene.add(groundMirror);

// создаём стекло
const geometryGlass = new THREE.PlaneGeometry(100, 100, 64, 64);
const materialGlass = new THREE.MeshPhysicalMaterial({  
 roughness: 0,
 transmission: 1,
 thickness: 2,
 side: THREE.DoubleSide,
 transparent: true,
 opacity: 0.8,
 clearcoat: 0,
 reflectivity: 0.1,
 specularIntensity: 0, 
 color: 0xcccccc
});
const floorGlass = new THREE.Mesh(geometryGlass, materialGlass);
floorGlass.position.y = -4.9;
floorGlass.rotateX(-Math.PI/2);
floorGlass.castShadow = true; 
floorGlass.receiveShadow = true;
scene.add(floorGlass);

// создаём стены
const wallGeometry = new THREE.PlaneGeometry(20, 12, 2, 2);
const wallMaterial = new THREE.MeshPhongMaterial({
  color: new THREE.Color('rgb(158,163,119)')
});
const wall = new THREE.Mesh(wallGeometry , wallMaterial);
wall.position.y = 1;
wall.castShadow = true; 
wall.receiveShadow = true;
scene.add(wall);

const wall2 = wall.clone();
wall2.rotation.y = Math.PI/2;
wall2.position.x = -10;
wall2.castShadow = true; 
wall2.receiveShadow = true;
scene.add(wall2);

const wall3 = wall.clone();
wall3.rotation.y = -Math.PI/2;
wall3.position.x = 10;
wall3.castShadow = true; 
wall3.receiveShadow = true;
scene.add(wall3);

// функция изменения размеров окна браузера
let resize = () => {
  camera.aspect = wrapper3Dw / wrapper3Dh;
  camera.updateProjectionMatrix();
  renderer.setSize(wrapper3Dw,wrapper3Dh);
  renderer.setPixelRatio(window.devicePixelRatio); 
}

// функция изменения размеров двери
let changeDoorSize = (dir) => {
  if (dir == 'more') {
    offset += delta;
  } else if (dir == 'less') {
    offset -= delta;
  }
  scene.getObjectByName('door').scale.set(0.05+offset,0.05+offset,0.05+offset);
}

let animate = () => {

  renderer.render(scene,camera);
  requestAnimationFrame(animate);

}; animate();

EventBus.$on('event name', (data) => {
    changeDoorSize(data.size);      
});

onMounted(() => {
    document.querySelector('#scene').appendChild(renderer.domElement);
    window.addEventListener('resize',resize);
});

</script>

<template>
  <div id='scene'></div>
</template>
