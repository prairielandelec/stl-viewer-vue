<template>
  <div ref="container" class="stl-viewer-container"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import * as THREE from 'three';
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'; 

const props = defineProps({
  model: {
    type: String,
    required: true
  },
  flipX: {
    type: Boolean,
    required: false
  }
  ,
  flipY: {
    type: Boolean,
    required: false
  }
  ,
  flipZ: {
    type: Boolean,
    required: false
  },
  zoomIn: {
    type: Number,
    required: false
  }

});

const container = ref(null);
let camera, scene, renderer, controls, animationFrameId;

const initThree = () => {
  if (!container.value) return;

  const width = container.value.clientWidth;
  const height = container.value.clientHeight;

  camera = new THREE.PerspectiveCamera(70, width / height, 1, 1000);
  scene = new THREE.Scene();
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(width, height);
  container.value.appendChild(renderer.domElement);

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableZoom = true;
  controls.autoRotate = true;
  controls.autoRotateSpeed = 1;

  scene.add(new THREE.HemisphereLight(0xffffff, 1.5));

  loadModel(props.model);

  window.addEventListener('resize', onWindowResize);
  animate();
};

const loadModel = (modelUrl) => {
  scene.traverse((object) => {
    if (object.isMesh) {
      scene.remove(object);
      object.geometry.dispose();
      object.material.dispose();
    }
  });

  new STLLoader().load(modelUrl, (geometry) => {
    const material = new THREE.MeshPhongMaterial({
      color: 0xFC6D09,
      specular: 100,
      shininess: 100,
    });
    const mesh = new THREE.Mesh(geometry, material);
    if (props.flipX)
    {
      mesh.rotation.x = (Math.PI / 2) * 90; //
    }
    if (props.flipY)
    {
      mesh.rotation.y = (Math.PI / 2); //
    }
    if (props.flipZ)
    {
      mesh.rotation.z = (Math.PI / 2); //
    }
    scene.add(mesh);

    const middle = new THREE.Vector3();
    geometry.computeBoundingBox();
    geometry.boundingBox.getCenter(middle);
    mesh.geometry.applyMatrix4(new THREE.Matrix4().makeTranslation(-middle.x, -middle.y, -middle.z));

    const largestDimension = Math.max(
      geometry.boundingBox.max.x - geometry.boundingBox.min.x,
      geometry.boundingBox.max.y - geometry.boundingBox.min.y,
      geometry.boundingBox.max.z - geometry.boundingBox.min.z
    );
    if (props.zoomIn)
    {
      camera.position.z = largestDimension / props.zoomIn;
    }
    else
    {
      camera.position.z = largestDimension * 1.5;
    }
    // Update controls target after model is loaded and centered
    controls.target.copy(mesh.position);
    controls.update();
  }, undefined, (error) => {
    console.error('An error occurred while loading the STL model:', error);
  });
};

const onWindowResize = () => {
  if (!container.value) return;
  const width = container.value.clientWidth;
  const height = container.value.clientHeight;
  renderer.setSize(width, height);
  camera.aspect = width / height;
  camera.updateProjectionMatrix();
};

const animate = () => {
  controls.update();
  renderer.render(scene, camera);
  animationFrameId = requestAnimationFrame(animate);
};

onMounted(() => {
  initThree();
});

onUnmounted(() => {
  window.removeEventListener('resize', onWindowResize);
  cancelAnimationFrame(animationFrameId);
  if (renderer) {
    renderer.dispose();
    renderer.domElement = null;
  }
  if (controls) {
    controls.dispose();
  }
});

watch(() => props.model, (newModel) => {
  if (newModel) {
    loadModel(newModel);
  }
});
</script>

<style scoped>
.stl-viewer-container {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
}
</style>
