<template>
  <div id="renderElement" class="renderer"></div>
</template>

<script setup lang="ts">
import * as THREE from 'three';
import * as CANNON from 'cannon-es';
import { onMounted } from 'vue';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 4;
camera.position.y = 2;
camera.rotation.x = -0.5;

const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(10, 10, 7.5);
scene.add(light);

const boardSize = { x: 4, y: 0.5, z: 5 };
const boardGeo = new THREE.BoxGeometry(boardSize.x, boardSize.y, boardSize.z);
const boardTexture = new THREE.TextureLoader().load('/wood.jpg');
const boardMat = new THREE.MeshBasicMaterial({ map: boardTexture });
const boardMesh = new THREE.Mesh(boardGeo, boardMat);

const boardGroup = new THREE.Group();
boardGroup.add(boardMesh);

const wallDefs = [
  { x: 0, y: 0.4, z: boardSize.z / 2, sx: boardSize.x, sy: 0.3, sz: 0.05 },
  { x: 0, y: 0.4, z: -boardSize.z / 2, sx: boardSize.x, sy: 0.3, sz: 0.05 },
  { x: boardSize.x / 2, y: 0.4, z: 0, sx: 0.05, sy: 0.3, sz: boardSize.z },
  { x: -boardSize.x / 2, y: 0.4, z: 0, sx: 0.05, sy: 0.3, sz: boardSize.z },
];

for (const { x, y, z, sx, sy, sz } of wallDefs) {
  const wallGeo = new THREE.BoxGeometry(sx, sy, sz);
  const wallMat = new THREE.MeshStandardMaterial({
    color: 0xadd8e6,
    transparent: true,
    opacity: 0.3,
    metalness: 0.1,
    roughness: 0.0,
    envMapIntensity: 1.0,
  });
  const wall = new THREE.Mesh(wallGeo, wallMat);
  wall.position.set(x, y, z);
  boardGroup.add(wall);
}
scene.add(boardGroup);

const ballRadius = 0.08;
const ballGeo = new THREE.SphereGeometry(ballRadius, 32, 32);
const ballMat = new THREE.MeshStandardMaterial({
  color: 0xeeeee,
  metalness: 1,
  roughness: 0.3,
});
const ballMesh = new THREE.Mesh(ballGeo, ballMat);
scene.add(ballMesh);

const world = new CANNON.World({ gravity: new CANNON.Vec3(0, -9.82, 0) });

const ballMaterial = new CANNON.Material({ restitution: 0.1 });
const wallMaterial = new CANNON.Material({ restitution: 0.0 });
// const contact = new CANNON.ContactMaterial(ballMaterial, wallMaterial, {
//   friction: 0.4,
//   restitution: 0.05,
//   contactEquationStiffness: 1e7,
//   contactEquationRelaxation: 4,
// });
// world.addContactMaterial(contact);

const boardBody = new CANNON.Body({ mass: 0, material: wallMaterial });
const boardShape = new CANNON.Box(
  new CANNON.Vec3(boardSize.x / 2, boardSize.y / 2, boardSize.z / 2),
);
boardBody.addShape(boardShape);

for (const { x, y, z, sx, sy, sz } of wallDefs) {
  const wallShape = new CANNON.Box(new CANNON.Vec3(sx / 2, sy / 2, sz / 2));
  boardBody.addShape(wallShape, new CANNON.Vec3(x, y, z));
}

world.addBody(boardBody);

const ballShape = new CANNON.Sphere(ballRadius);
const ballBody = new CANNON.Body({
  mass: 1,
  shape: ballShape,
  position: new CANNON.Vec3(-1.7, 1, 2.2),
  material: ballMaterial,
  linearDamping: 0.3,
  angularDamping: 0.3,
});
world.addBody(ballBody);

function syncMeshToBody(mesh: THREE.Object3D, body: CANNON.Body) {
  mesh.position.set(body.position.x, body.position.y, body.position.z);
  mesh.quaternion.set(body.quaternion.x, body.quaternion.y, body.quaternion.z, body.quaternion.w);
}

function animate() {
  world.step(1 / 60);
  syncMeshToBody(boardGroup, boardBody);
  syncMeshToBody(ballMesh, ballBody);
  renderer.render(scene, camera);
}
renderer.setAnimationLoop(animate);

const force = 4;

onMounted(() => {
  window.addEventListener('keydown', (e) => {
    switch (e.key.toLowerCase()) {
      case 'w':
        ballBody.applyForce(new CANNON.Vec3(0, 0, -force), ballBody.position);
        break;
      case 's':
        ballBody.applyForce(new CANNON.Vec3(0, 0, force), ballBody.position);
        break;
      case 'a':
        ballBody.applyForce(new CANNON.Vec3(-force, 0, 0), ballBody.position);
        break;
      case 'd':
        ballBody.applyForce(new CANNON.Vec3(force, 0, 0), ballBody.position);
        break;
      case ' ':
        ballBody.velocity.set(0, 0, 0);
        ballBody.angularVelocity.set(0, 0, 0);
        break;
    }
  });
});
</script>

<style scoped>
body {
  margin: 0;
  overflow: hidden;
}
canvas {
  display: block;
}
</style>
