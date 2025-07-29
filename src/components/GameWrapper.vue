<template></template>

<script setup lang="ts">
import * as THREE from 'three';
import * as CANNON from 'cannon-es';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 5;

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
const boardEdges = new THREE.EdgesGeometry(boardGeo);
const boardWireframe = new THREE.LineSegments(
  boardEdges,
  new THREE.LineBasicMaterial({ color: 0x000000 }),
);

boardMesh.add(boardWireframe);
scene.add(boardMesh);

const ballRadius = 0.15;
const ballGeo = new THREE.SphereGeometry(ballRadius, 32, 32);
const ballMat = new THREE.MeshStandardMaterial({ color: 0xaaaaaa, metalness: 1, roughness: 0.3 });
const ballMesh = new THREE.Mesh(ballGeo, ballMat);
scene.add(ballMesh);

const world = new CANNON.World({ gravity: new CANNON.Vec3(0, -9.82, 0) });

const boardShape = new CANNON.Box(
  new CANNON.Vec3(boardSize.x / 2, boardSize.y / 2, boardSize.z / 2),
);
const boardBody = new CANNON.Body({
  mass: 0,
  shape: boardShape,
  position: new CANNON.Vec3(0, 0, 0),
});
boardBody.quaternion.x += 0.25;
world.addBody(boardBody);
boardMesh.quaternion.set(
  boardBody.quaternion.x,
  boardBody.quaternion.y,
  boardBody.quaternion.z,
  boardBody.quaternion.w,
);

const ballShape = new CANNON.Sphere(ballRadius);
const ballBody = new CANNON.Body({
  mass: 1,
  shape: ballShape,
  position: new CANNON.Vec3(0, 2, 0),
  material: new CANNON.Material({ restitution: 0.6 }),
});
world.addBody(ballBody);

const boardRotation = boardBody.quaternion.clone();

const wallConfigs = [{ width: 4, height: 0.3, depth: 0.2, x: 0, y: 0.4, z: 0.3 }];

for (const { width, height, depth, x, y, z } of wallConfigs) {
  const wallGeo = new THREE.BoxGeometry(width, height, depth);
  const wallMat = new THREE.MeshStandardMaterial({ color: 0xffff });
  const wallMesh = new THREE.Mesh(wallGeo, wallMat);
  wallMesh.position.set(x, y, z);
  wallMesh.quaternion.copy(boardRotation);
  scene.add(wallMesh);

  const wallShape = new CANNON.Box(new CANNON.Vec3(width / 2, height / 2, depth / 2));
  const wallBody = new CANNON.Body({
    mass: 0,
    shape: wallShape,
    position: new CANNON.Vec3(x, y, z),
  });
  wallBody.quaternion.copy(boardRotation);
  world.addBody(wallBody);
}

function syncMeshToBody(mesh: THREE.Object3D, body: CANNON.Body) {
  mesh.position.set(body.position.x, body.position.y, body.position.z);
  mesh.quaternion.set(body.quaternion.x, body.quaternion.y, body.quaternion.z, body.quaternion.w);
}

function animate() {
  world.step(1 / 60);
  syncMeshToBody(ballMesh, ballBody);
  renderer.render(scene, camera);
}
renderer.setAnimationLoop(animate);
</script>

<style scoped></style>
