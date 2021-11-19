<template>
  <div class="Create"></div>
  <!-- Draw Wall -->
  <div v-if="debugChoose == 1" class="debug-board">
    <n-space vertical>
      <n-card title="Wall Height" size="medium">
        <n-space vertical>
          <n-slider v-model:value="wallHeight" :step="1" :min="10" :max="20" />
          <n-input v-model:value="wallHeight" type="number" />
        </n-space>
      </n-card>
      <n-card title="orbitcontrol" size="medium">
        <n-space vertical>
          enable
          <n-switch v-model:value="orbitSwitch" @update:value="handleControl" />reset
          <n-switch v-model:value="orbitReset" @update:value="handleControlReset" />
        </n-space>
      </n-card>
      <n-card title="next step" size="medium">
        <n-space vertical>
          <n-button type="primary" size="medium" @click="handleDebugChoose(1)">next step</n-button>
        </n-space>
      </n-card>
    </n-space>
  </div>
  <!-- Position Object -->
  <div v-if="debugChoose == 2" class="debug-board">
    <n-space vertical>
      <n-card title="Upload - File" size="medium">
        <n-upload @before-upload="beforeUpload" action>
          <n-upload-dragger>
            <div style="margin-bottom: 12px;">
              <n-icon size="20" :depth="3">
                <archive-icon />
              </n-icon>
            </div>
            <n-text style="font-size: 16px;">点击或者拖动文件到该区域来上传</n-text>
            <n-p depth="3" style="margin: 8px 0 0 0;">支持GLB, GLTF格式</n-p>
          </n-upload-dragger>
        </n-upload>
      </n-card>
      <n-card title="orbitcontrol" size="medium">
        <n-switch v-model:value="orbitSwitch" @update:value="handleControl" />
      </n-card>
    </n-space>
  </div>
  <div v-if="hasSelect" class="debug-board-right">
    <n-space vertical>
      <n-card title="Mode" size="medium">
        <n-space vertical>
          <n-select v-model:value="Mode" :options="Modeoptions" @update:value="changeMode" />
          <n-button type="error" size="medium" @click="clearSelection">Clear Selection</n-button>
        </n-space>
      </n-card>
      <n-card title="Translate" size="medium">
        <n-space vertical>
          <n-slider
            v-model:value="INTERSECTED.position.x"
            :step="1"
            :min="-length * 50"
            :max="length * 50"
            @update:value="handletranslateX"
          />
          <n-slider
            v-model:value="INTERSECTED.position.y"
            :step="1"
            :min="-length * 50"
            :max="length * 50"
            @update:value="handletranslatey"
          />
          <n-slider
            v-model:value="INTERSECTED.position.z"
            :step="1"
            :min="-length * 50"
            :max="length * 50"
            @update:value="handletranslateZ"
          />
        </n-space>
      </n-card>
      <n-card title="Rotate" size="medium">
        <n-space vertical>
          <n-slider
            v-model:value="INTERSECTED.rotation.x"
            :step="Math.PI / 90"
            :min="-Math.PI"
            :max="Math.PI"
            @update:value="handlerotateX"
          />
          <n-slider
            v-model:value="INTERSECTED.rotation.y"
            :step="Math.PI / 90"
            :min="-Math.PI"
            :max="Math.PI"
            @update:value="handlerotatey"
          />
          <n-slider
            v-model:value="INTERSECTED.rotation.z"
            :step="Math.PI / 90"
            :min="-Math.PI"
            :max="Math.PI"
            @update:value="handlerotateZ"
          />
        </n-space>
      </n-card>
      <n-card title="Scale" size="medium">
        <n-space vertical>
          <n-slider
            v-model:value="INTERSECTED.scale.x"
            :step="0.1"
            :min="0"
            :max="100"
            @update:value="handlescaleX"
          />
          <n-slider
            v-model:value="INTERSECTED.scale.y"
            :step="0.1"
            :min="0.1"
            :max="100"
            @update:value="handlescaley"
          />
          <n-slider
            v-model:value="INTERSECTED.scale.z"
            :step="0.1"
            :min="0.1"
            :max="100"
            @update:value="handlescaleZ"
          />
          <n-button type="primary" size="medium" @click="scaleLarge">Larger</n-button>
          <n-button type="primary" size="medium" @click="scaleSmall">Smaller</n-button>
        </n-space>
      </n-card>
    </n-space>
  </div>
</template>
<script setup>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { TransformControls } from 'three/examples/jsm/controls/TransformControls'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js'
import * as dat from "dat.gui";
import Stats from 'three/examples/jsm/libs/stats.module'
import { NButton, NCard, NInput, NSlider, NSpace, NSwitch, NUpload, NUploadDragger, NIcon, NText, NP, NCheckboxGroup, NCheckbox, NSelect } from 'naive-ui'
import { ArchiveOutline as ArchiveIcon } from '@vicons/ionicons5'
import { ref } from 'vue'
let scene, camera, renderer, controls;
let transformcontrols
let INTERSECTED = ref(null)
let objs = []
let tmparray = [1]
// 是否选中了物体
// 编辑物体属性
const Mode = ref('tranlate')
const Modeoptions = [
  {
    label: 'translate',
    value: 'translate'
  },
  {
    label: 'rotate',
    value: 'rotate'
  },
  {
    label: 'scale',
    value: 'scale'
  }
]
const changeMode = (mode) => {
  transformcontrols.setMode(mode)
}
const hasSelect = ref(false)
const handletranslateX = (value) => {
  INTERSECTED.value.position.x = value
}
const handletranslateY = (value) => {
  INTERSECTED.value.position.y = value
}
const handletranslateZ = (value) => {
  INTERSECTED.value.position.z = value
}

const handlerotateX = (value) => {
  INTERSECTED.value.rotation.x = value
}
const handlerotateY = (value) => {
  INTERSECTED.value.rotation.y = value
}
const handlerotateZ = (value) => {
  INTERSECTED.value.rotation.z = value
}

const handlescaleX = (value) => {
  INTERSECTED.value.scale.x = value
}
const handlescaleY = (value) => {
  INTERSECTED.value.scale.y = value
}
const handlescaleZ = (value) => {
  INTERSECTED.value.scale.z = value
}

const scaleLarge = (e) => {
  INTERSECTED.value.scale.x += 2
  INTERSECTED.value.scale.y += 2
  INTERSECTED.value.scale.z += 2
}

const scaleSmall = (e) => {
  INTERSECTED.value.scale.x -= 1
  INTERSECTED.value.scale.y -= 1
  INTERSECTED.value.scale.z -= 1
}

const debugChoose = ref(1)
const wallHeight = ref(15)
// 上传文件
const gltfloader = new GLTFLoader();
const objloader = new OBJLoader()
const uploadFile = ref(null)
const beforeUpload = ({ file, fileList }) => {
  file.status = 'finished'
  loadFile(file.file)
}

const loadFile = (file) => {
  if (file.name.includes('.glb') || file.name.includes('.gltf')) {
    let reader = new FileReader();
    reader.readAsArrayBuffer(file);
    reader.onload = gltfText => {
      gltfloader.parse(gltfText.target.result, '', (gltfData) => {
        scene.add(gltfData.scene)
      },
        errMassage => { console.error(errMassage) })
    }
  } else if (file.name.includes('.obj')) {
    let reader = new FileReader();
    reader.readAsArrayBuffer(file);
    reader.onload = objText => {
      objloader.parse(objText.target.result, '', (objData) => {
        scene.add(objData.scene)
      },
        errMassage => { console.error(errMassage) })
    }
  }

}

// 模板文件
const loadTemplate = (source) => {
  // loader.load('/models/Duck/glTF-Binary/Duck.glb', gltf => {
  //   console.log(gltf)
  //   const duck = gltf.scene.children[0]
  //   scene.add(duck)
  // })
  gltfloader.load(source, gltf => {
    const mesh = gltf.scene.children[0]
    scene.add(mesh)
  })
}


// let precolor
// 选择物体
const selectObj = (event) => {
  // console.log(INTERSECTED)
  let intersects = getIntersects(event);
  if (intersects.length && intersects[0].object.type == 'Mesh' && intersects[0].object.name !== ''
    && intersects[0].object.name !== 'X'
    && intersects[0].object.name !== 'Y'
    && intersects[0].object.name !== 'Z') {
    if (INTERSECTED.value !== intersects[0].object) {
      INTERSECTED.value = intersects[0].object
      const params = {};
      params.scaleAll = 1
      hasSelect.value = true
      if (INTERSECTED.value.material.emissive) {
        INTERSECTED.value.material.emissive.setHex(0x001aff);
      }
      transformcontrols.attach(INTERSECTED.value)

      scene.add(transformcontrols)
    }
    window.removeEventListener('mousedown', selectObj, false);
  }
}

const clearSelection = () => {
  INTERSECTED.value.material.emissive.setHex(0x00000);
  INTERSECTED.value = null
  scene.remove(transformcontrols)
  window.addEventListener('mousedown', selectObj, false);
}

// const addGUI = (obj,params) => {
//   const gui = new dat.GUI();
//   console.log(obj)
//   gui.add(obj.position, "x").min(-5).max(100).step(1).name("Position X");
//   gui.add(obj.position, "y").min(-5).max(100).step(1).name("Position Y");
//   gui.add(obj.position, "z").min(-5).max(100).step(1).name("Position Z");
//   gui.add(obj.rotation, "x").min(-Math.PI).max(Math.PI).step(Math.PI/90).name("Rotation X");
//   gui.add(obj.rotation, "y").min(-Math.PI).max(Math.PI).step(Math.PI/90).name("Rotation Y");
//   gui.add(obj.rotation, "z").min(-Math.PI).max(Math.PI).step(Math.PI/90).name("Rotation Z");
//   gui.add(obj.scale, "x").min(0).max(100).step(0.1).name("Scale X");
//   gui.add(obj.scale, "y").min(0).max(100).step(0.1).name("Scale Y");
//   gui.add(obj.scale, "z").min(0).max(100).step(0.1).name("Scale Z");
//   gui.add(params, "scaleAll").min(0).max(100).step(0.1).name("Scale All").onChange(rebuildObj(obj,params))
// }

// const rebuildObj = (obj, params) => {
//   console.log(params)
//   obj.scale.set(params.scaleAll, params.scaleAll, params.scaleAll)
// }


// 切换面板
const handleDebugChoose = (value) => {
  debugChoose.value = value + 1
  if (value == 1) {
    orbitSwitch.value = true
    handleControl(true)
    scene.remove(groupPoints)
    window.removeEventListener('mousedown', onMouseDown, false);
    // window.removeEventListener('mousemove', onMouseMove, false);
    window.addEventListener('mousedown', selectObj, false);

  }
}

// 视角重置
const orbitReset = ref(false)
const handleControlReset = (value) => {
  controls.reset()
  orbitSwitch.value = false
  handleControl(false)
  // scene.remove(groupPoints)
}

// open orbitControl
const orbitSwitch = ref(false)
const handleControl = (value) => {
  controls.enabled = value
}

// 性能插件
// const initStats = () => {

//   let stats = new Stats();

//   stats.setMode(0);
//   stats.domElement.style.position = 'absolute';
//   stats.domElement.style.left = '0px';
//   stats.domElement.style.bottom = '0px';
//   document.body.appendChild(stats.dom)
//   return stats;

// }
// let stats = initStats();

// 地面网格所需变量
let length = 200; // 地面尺寸

// 场景
const initScene = () => {

  scene = new THREE.Scene();

}

// 相机
const initCamera = () => {

  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10000);
  camera.position.set(0, 300, 0);
  camera.lookAt(new THREE.Vector3(0, 0, 0));

}

// 渲染器
const initRender = () => {
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

}

// 灯光
const initLight = () => {

  let ambientLight = new THREE.AmbientLight(0x333333);
  scene.add(ambientLight);

  let directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(100, 300, 200);
  scene.add(directionalLight);

}

// 控制器
const initControls = () => {
  controls = new OrbitControls(camera, renderer.domElement);
  // controls.minAzimuthAngle = Math.PI;
  // controls.maxAzimuthAngle = Math.PI;
  transformcontrols = new TransformControls(camera, renderer.domElement)
  transformcontrols.addEventListener('mouseDown', function () {
    orbitSwitch.value = false
    handleControl(false)
  });
  transformcontrols.addEventListener('mouseUp', function () {
    orbitSwitch.value = true
    handleControl(true)
  });
  controls.enableDamping = true
  controls.saveState()
  controls.enabled = false
}

// 场景内容
const initContent = () => {
  const geometry = new THREE.BufferGeometry()
  let lineMaterial = new THREE.LineBasicMaterial({ color: 0x808080 });/* 基础线材质 */
  let pointsArray = []

  let planeGeometry = new THREE.PlaneGeometry(length, 10);/* 平面 width：200,、height：10 */
  let planeMaterial = new THREE.MeshBasicMaterial({ color: 0xD9D9D9, side: THREE.DoubleSide });/* 平面材质 */

  pointsArray.push(new THREE.Vector3(-length / 2, 0, 0));/* 顶点(-100, 0, 0) */
  pointsArray.push(new THREE.Vector3(length / 2, 0, 0)); /* 顶点( 100, 0, 0) */

  geometry.setFromPoints(pointsArray)
  /* 循环创建线段 */
  for (let i = 0; i <= length / 10; i++) {

    /* 横向线段 */
    let lineX = new THREE.Line(geometry, lineMaterial);
    lineX.position.z = (i * 10) - length / 2;
    scene.add(lineX);

    /* 纵向线段 */
    let lineY = new THREE.Line(geometry, lineMaterial);
    lineY.rotation.y = 0.5 * Math.PI;
    lineY.position.x = (i * 10) - length / 2;
    scene.add(lineY);

  }

  /* 创建包围平面 */
  let planeX_left = new THREE.Mesh(planeGeometry, planeMaterial);
  planeX_left.rotation.y = 0.5 * Math.PI;
  planeX_left.position.x = -length / 2;

  let planeX_right = planeX_left.clone();
  planeX_right.position.x = length / 2;

  let planeY_top = new THREE.Mesh(planeGeometry, planeMaterial);
  planeY_top.position.z = -length / 2;

  let planeY_bottom = planeY_top.clone();
  planeY_bottom.position.z = length / 2;

  scene.add(planeY_bottom);
  scene.add(planeY_top);
  scene.add(planeX_left);
  scene.add(planeX_right);

  /* 四个包围面的位置 y轴向上5 */
  scene.traverse(function (object) {

    if (object.isMesh) {

      if (object.geometry.type === 'PlaneGeometry') {

        object.position.y = 5;

      }

    }

  });

}

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

let pointsArray = []
let window_mouse = true

const getIntersects = (event) => {
  let raycaster = new THREE.Raycaster();
  let mouse = new THREE.Vector2();

  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  // let normal = new THREE.Vector3(0, 1, 0);
  /* 创建平面 */
  // let planeGround = new THREE.Plane(normal, 0);
  // console.log(mouse,camera)
  /* 从相机发出一条射线经过鼠标点击的位置 */
  raycaster.setFromCamera(mouse, camera);

  /* 计算相机到射线的对象，可能有多个对象，返回一个数组，按照距离相机远近排列 */
  let intersects = raycaster.intersectObjects(scene.children);

  /* 返回向量 */
  return intersects;

}

const planeBuilder = (point1, point2, wallHeight) => {
  wallHeight = 20
  let geometry = new THREE.BufferGeometry();
  let vertices = new Float32Array([
    point1.x, 0.01, point1.z,
    point2.x, 0.01, point2.z,
    point2.x, wallHeight, point2.z,

    point2.x, wallHeight, point2.z,
    point1.x, wallHeight, point1.z,
    point1.x, 0.01, point1.z,
  ]);

  geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
  let material = new THREE.MeshBasicMaterial({ color: 0xd9d9d9, side: THREE.DoubleSide });
  let mesh = new THREE.Mesh(geometry, material);
  console.log(mesh)
  scene.add(mesh)
}

const groupPoints = new THREE.Group()
const onMouseDown = (event) => {

  /* 获取相机发出的射线与 Plane 相交点*/
  let intersects = getIntersects(event);
  if (intersects.length == 2) {
    intersects = intersects[0].point
    /* 存放网格的三维坐标 */
    let vector3_x, vector3_z;

    /* 鼠标左键按下时，创建点和线段 */
    if (event.button === 0) {

      if (!window_mouse) {

        window.addEventListener('mousemove', onMouseMove, false);

        /* 依据 windwo_mouse 标识避免事件的重复添加 */
        window_mouse = true;

      }
      // console.log(intersects.x, intersects.z)
      intersects.x = Math.round(intersects.x)
      intersects.y = Math.round(intersects.y)
      /* 判断交点是否在 x(-100, 100) ，z(-100, 100)(平面)之间 */
      if (Math.abs(intersects.x) < length / 2 && Math.abs(intersects.z) < length / 2) {
        /* 若交点此时在平面之内则创建点（Points） */
        let pointsGeometry = new THREE.BufferGeometry();
        let intersectArray = []
        intersectArray.push(intersects);
        pointsGeometry.setFromPoints(intersectArray)
        let pointsMaterial = new THREE.PointsMaterial({ color: 0xff0000, size: 3 });
        let points = new THREE.Points(pointsGeometry, pointsMaterial);

        pointsArray.push(intersects);

        /* 创建线段 */
        if (pointsArray.length >= 2) {
          // console.log(wallHeight)
          planeBuilder(pointsArray[0], pointsArray[1], wallHeight.value)
          pointsArray.shift()
        }
        groupPoints.add(points)
        scene.add(groupPoints);

      }

    }

    /* 鼠标右键按下时 回退到上一步的点，并中断绘制 */
    if (event.button === 2) {

      window.removeEventListener('mousemove', onMouseMove, false);

      /* 移除事件之后，要设置为 false 为了避免事件的重复添加 */
      window_mouse = false;

      /* 鼠标左键未点击时线段的移动状态 */
      if (scene.getObjectByName('line_move')) {

        scene.remove(scene.getObjectByName('line_move'));

        /* 删除数组中的元素，否则的话再次重绘会链接之前的点接着重绘 */
        pointsArray.shift();

      }

    }
  }

}

// const onMouseMove = (event) => {

//   var intersects = getIntersects(event);

//   /* 判断交点是否在 x(-100, 100) ，z(-100, 100)(平面)之间 */
//   if (Math.abs(intersects.x) < length / 2 && Math.abs(intersects.z) < length / 2) {

//     /* 鼠标左键未点击时线段的移动状态 */
//     if (scene.getObjectByName('line_move')) {

//       scene.remove(scene.getObjectByName('line_move'));

//     }
//     /* 创建线段 */
//     var lineGeometry = new THREE.Geometry();
//     var lineMaterial = new THREE.LineBasicMaterial({ color: 0x00ff00 });

//     if (pointsArray.length > 0) {

//       lineGeometry.vertices.push(pointsArray[0].geometry.vertices[0]);

//       var mouseVector3 = new THREE.Vector3(intersects.x, 0, intersects.z);

//       lineGeometry.vertices.push(mouseVector3);

//       var line = new THREE.Line(lineGeometry, lineMaterial);
//       line.name = 'line_move';

//       scene.add(line);

//     }

//   }

// }


/* 更新数据 */
const update = () => {
  // stats.update();
  controls.update();
}

/* 窗口自动适应 */
const onWindowResize = () => {

  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);

}

/* 循环调用 */
const animate = () => {
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(scene.children);
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
  update();
}

/* 初始化 */
const init = () => {
  /* 兼容性判断 */
  //if (!Detector.webgl) Detector.addGetWebGLMessage();

  initScene();
  initCamera();
  initRender();
  initLight();
  initContent();
  initControls();

  /* 事件监听 */
  window.addEventListener('resize', onWindowResize, false);
  // window.addEventListener('mousemove', onMouseMove, false);
  window.addEventListener('mousedown', onMouseDown, false);/* 使用mousedown的时候可以判断出点击的鼠标左右键之分 */
  // window.addEventListener('keydown', onKeyDown, false);/* 使用事件的时候要把前面的on给去掉 */

}

/* 初始加载 */
init()
animate()
</script>
<style scoped>
* {
  margin: 0;
}
.webgl {
  position: fixed;
  top: 0;
  left: 0;
  outline: none;
}
.debug-board:hover {
  border: none;
  width: 400px;
  height: 700px;
  margin-top: 40px;
  position: absolute;
  border-radius: none;
  right: 40px;
}
.debug-board {
  color: #000000;
  border: 1px solid white;
  width: 20px;
  height: 700px;
  margin-top: 40px;
  position: absolute;
  right: 40px;
  border-radius: 10px;
  overflow: hidden;
  transition: all 0.8s;
}
.debug-board-right {
  color: #000000;
  border: 1px solid white;
  width: 20px;
  height: 800px;
  margin-top: 40px;
  position: absolute;
  left: 40px;
  border-radius: 10px;
  overflow: hidden;
  transition: all 0.8s;
}
.debug-board-right:hover {
  border: none;
  width: 400px;
  height: 800px;
  margin-top: 40px;
  position: absolute;
  border-radius: none;
  right: 40px;
}
#gui {
  position: absolute;
  top: 20px;
  left: 20px;
}
</style>
