<template>
  <div id="canvas">
    <div class="controls">
      <h1>Floors {{ floorCount }}</h1>
      <button @click="modifyFloorCount()">Insert Floor</button>
      <button @click="modifyFloorCount(false)">Remove Floor</button>
      <div
        style="display: grid; grid-template-columns: repeat(3, 1fr); justify-content: space-between; gap: 0.5em; margin: 1em 0;">
        <div style="display: flex; align-items: center; justify-content: start; gap: 1em;">
          <label for="">Lift 1</label>
          <input v-model="startLift1" type="checkbox" name="" id="">
        </div>
        <div style="display: flex; align-items: center; justify-content: start;gap: 1em;">
          <label for="">Lift</label>
          <input v-model="startLift2" type="checkbox" name="" id="">
        </div>
        <div style="display: flex; align-items: center; justify-content: start;gap: 1em;">
          <label for="">Jump Lift</label>
          <input v-model="startJumpLift" type="checkbox" name="" id="">
        </div>
      </div>
    </div>
  </div>
</template>


<script setup lang="ts" allowJs:true>
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { onMounted, ref, watch } from 'vue';


const floorCount = ref<number>(0)
const floorsList = ref<THREE.Mesh[]>([])
const AddonsSegments = ref<THREE.Mesh[]>([])
const startLift1 = ref<boolean>(false);
const startLift2 = ref<boolean>(false);
const startJumpLift = ref<boolean>(false);

const clock = new THREE.Clock();
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

camera.position.x = 20;

const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);

const controls = new OrbitControls(camera, renderer.domElement);


camera.position.set(0, 8, 0)
controls.target.set(0, 8, 0)
controls.update();



const directionalLight = new THREE.DirectionalLight(0xffffff, 2);
scene.add(directionalLight);


const light = new THREE.AmbientLight(0x404040, 10); // soft white light
scene.add(light);

const size = 14;
const divisions = 14;

const gridHelper = new THREE.GridHelper(size, divisions);
scene.add(gridHelper);

function CreateFloor(floors: number) {
  const geometry = new THREE.BoxGeometry(6, 1, 6);
  const material = new THREE.MeshPhysicalMaterial({
    color: 0x4f9fdd,
    wireframe: false,
    transmission: 0.148,
    transparent: true,
    opacity: 0.15,
    metalness: 0.2,
    roughness: 0.8,
    ior: 1.5,
    specularIntensity: 0.1,
    envMapIntensity: 0.06,
    side: THREE.DoubleSide,
    depthWrite: false,
  });

  const cube = new THREE.Mesh(geometry, material);
  cube.position.set(0, floors + 0.5 - 1, 0);

  scene.add(cube);
  floorsList.value.push(cube)
}

function AddShaft(floors: number, x: number, z: number, color: number) {
  const liftGeometry = new THREE.BoxGeometry(1, 1, 1);
  const liftMaterial = new THREE.MeshPhysicalMaterial({
    color: color,
    wireframe: false,
    transmission: 0.148,
    transparent: true,
    opacity: 0.15,
    metalness: 0.2,
    roughness: 0.8,
    ior: 1.5,
    specularIntensity: 0.1,
    envMapIntensity: 0.06,
    side: THREE.DoubleSide,
    depthWrite: false,
  });

  const cube = new THREE.Mesh(liftGeometry, liftMaterial);
  cube.position.set(x, floors + 0.5 - 1, z);
  scene.add(cube);

  AddonsSegments.value.push(cube)
}


function AddHoist(floors: number, x: number, z: number, color: number) {
  const liftGeometry = new THREE.BoxGeometry(1, 1, 1);
  const liftMaterial = new THREE.MeshPhysicalMaterial({
    color: color,
    wireframe: false,
    transmission: 0.148,
    transparent: true,
    opacity: 0.5,
    metalness: 0.5,
    roughness: 0.8,
    ior: 1.5,
    specularIntensity: 0.1,
    envMapIntensity: 0.06,
    side: THREE.DoubleSide,
    depthWrite: false,
  });

  const liftCube = new THREE.Mesh(liftGeometry, liftMaterial);
  liftCube.position.set(x, floors + 0.5 - 1, z);
  scene.add(liftCube);

  AddonsSegments.value.push(liftCube)
}


function AddJumpLift(floors: number, x: number, z: number, color: number) {
  const liftGeometry = new THREE.BoxGeometry(0.1, 1, 0.1);
  const liftMaterial = new THREE.MeshPhysicalMaterial({
    color: color,
    wireframe: false,
    transmission: 0.148,
    transparent: true,
    opacity: 0.5,
    metalness: 0.5,
    roughness: 0.8,
    ior: 1.5,
    specularIntensity: 0.1,
    envMapIntensity: 0.06,
    side: THREE.DoubleSide,
    depthWrite: false,
  });

  const liftCube = new THREE.Mesh(liftGeometry, liftMaterial);
  liftCube.position.set(x, floors + 0.5 - 1, z);
  scene.add(liftCube);
  AddonsSegments.value.push(liftCube)
}




function deleteFloor(floors: number) {

  const floorToDelete = scene.getObjectByProperty('uuid', floorsList.value[floors - 1].uuid)
  scene.remove(floorToDelete as THREE.Mesh)


  AddonsSegments.value.forEach(item => {
    if (item.position.y == (floors - 0.5)) {
      scene.remove(scene.getObjectByProperty('uuid', item.uuid) as THREE.Mesh)
    }
  })

  floorsList.value.pop();
  AddonsSegments.value.pop();
  AddonsSegments.value.pop();
  AddonsSegments.value.pop();
}



const lift = new THREE.Mesh(new THREE.BoxGeometry(0.5, 1, 0.5), new THREE.MeshPhysicalMaterial({
  color: 0x4FC295
}))

const lift1 = new THREE.Mesh(new THREE.BoxGeometry(0.5, 1, 0.5), new THREE.MeshPhysicalMaterial({
  color: 0x4FC295
}))

const jumplift = new THREE.Mesh(new THREE.BoxGeometry(1, 1, 2), new THREE.MeshPhysicalMaterial({
  color: 0xff7e38,
  wireframe: false,
  transmission: 0.148,
  transparent: true,
  opacity: 0.5,
  metalness: 0.5,
  roughness: 0.8,
  ior: 1.5,
  specularIntensity: 0.1,
  envMapIntensity: 0.06,
  side: THREE.DoubleSide,
  depthWrite: false,
}))

lift.position.set(-0.5, 1 / 2, 0.5);
lift1.position.set(-1.5, 1 / 2, 0.5);
jumplift.position.set(-3.5, 1 / 2, 1.5);



const modifyFloorCount = (ShouldIncrease: boolean = true) => {
  if (ShouldIncrease == true)
    floorCount.value++;
  else {
    if (floorCount.value > 0) {
      floorCount.value--;
    }
  }
}

watch(floorCount, (newVal: any, oldVal: any) => {
  if (newVal > oldVal) {
    if (oldVal >= 0) {
      camera.position.z += newVal * 0.15;
      CreateFloor(newVal as number);
      AddShaft(newVal as number, -1.5, 0.5, 0x4FC295);
      AddShaft(newVal as number, -0.5, 0.5, 0x4FC295);
      AddHoist(newVal as number, 3.5, 1.5, 0x1350F5);
      AddJumpLift(newVal as number, -3, 1.5, 0xff7e38);
      console.log(AddonsSegments.value)
    }
  } else {
    if (oldVal > 0) {
      camera.position.z -= newVal * 0.15;
      deleteFloor(oldVal);
    }
  }

  if (floorCount.value > 1) {
    scene.add(lift)
    scene.add(lift1)
    scene.add(jumplift)
  } else {
    scene.remove(lift)
    scene.remove(lift1)
    scene.remove(jumplift)
  }

})

onMounted(() => {
  camera.position.z = 15;
  (document.getElementById('canvas') as HTMLElement).appendChild(renderer.domElement);


  function animate() {
    const elapsedTime = clock.getElapsedTime()

    if (floorCount.value > 3) {
      if (startLift1.value == true)
        lift.position.y = ((floorCount.value * Math.sin(elapsedTime)) / 2) + floorCount.value / 2 + 0.5;

      if (startLift2.value == true)
        lift1.position.y = ((floorCount.value * Math.cos(elapsedTime)) / 2) + floorCount.value / 2 + 0.5;

      if (startJumpLift.value == true)
        jumplift.position.y = ((floorCount.value * Math.cos(elapsedTime)) / 2) + floorCount.value / 2 + 0.5;
    }

    controls.update();
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
  }

  animate();
})
</script>

<style scoped>
.controls {
  padding: 1em;
  width: 300px;
  background: white;
  position: fixed;
  border-radius: 5px;
  top: 0;
  right: 0;
  margin: 1em;
  display: flex;
  flex-direction: column;
}

button {
  padding: 1em;
  width: 100%;
  outline: 0;
  border: 0;
  background-color: rgb(0, 95, 183);
  color: white;
  border-radius: 3px;
  font-weight: bold;
  cursor: pointer;
  margin: 5px 0;
}
</style>