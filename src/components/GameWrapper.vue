<template></template>
<script setup lang="ts">
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();

renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const textureLoader = new THREE.TextureLoader();
const woodTexture = textureLoader.load('/wood.jpg');

const geometry = new THREE.BoxGeometry(4, 0.5, 5);
const material = new THREE.MeshBasicMaterial({ map: woodTexture });
const cube = new THREE.Mesh(geometry, material);
const edges = new THREE.EdgesGeometry(geometry);
const line = new THREE.LineSegments(edges, new THREE.LineBasicMaterial({ color: 0x00000 }));
cube.add(line);
cube.rotation.x += 0.6;

scene.add(cube);
camera.position.z = 5;

function animate() {
  renderer.render(scene, camera);
}

renderer.setAnimationLoop(animate);
</script>
<style scoped></style>
