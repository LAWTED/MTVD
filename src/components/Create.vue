<template>
  <div class="Create">
  </div>
</template>
<script>
import * as THREE from 'Three'
import { OrbitControls } from 'Three/examples/jsm/controls/OrbitControls'
export default {
  name: 'Create',
  data () {
    return {}
  },
  setup () {
    // 1、创建场景和摄像机
    const scene = new THREE.Scene()
    // 2、创建摄像机 PerspectiveCamera('视角', '指定投影窗口长宽比', '从距离摄像机多远开始渲染', '截止多远停止渲染100')
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 2, 1000)
    // 3、创建THREEJs 渲染器
    const renderer = new THREE.WebGLRenderer({ antialias: true })
    // 设置渲染器场景大小
    renderer.setSize(window.innerWidth, window.innerHeight)
    document.body.appendChild(renderer.domElement)

    // const plane = new THREE.Plane( new THREE.Vector3( 1, 1, 0.2 ), 3 );
    // const helper = new THREE.PlaneHelper( plane, 1, 0xffff00 );
    // scene.add( helper );

    // 创建几何模型 BoxGeometry('x轴', '轴', 'z轴')
    const boxGeometry = new THREE.BoxBufferGeometry(1, 1, 1)
    const planeGeometry = new THREE.PlaneGeometry( 10, 10, 20, 20);
    const planeMaterial = new THREE.MeshBasicMaterial( {color: 0xffff00, side: THREE.DoubleSide} );
    planeMaterial.wireframe = true
    const plane = new THREE.Mesh( planeGeometry, planeMaterial );
    plane.rotation.x = Math.PI/2
    scene.add( plane );
    // 创建纹理贴图  前后  上下  左右

    // 添加材质
    const boxMaterial = new THREE.MeshBasicMaterial( {color: 0x00ff00})

    // 创建网格对象
    const cube = new THREE.Mesh(boxGeometry, boxMaterial)
    cube.position.y = cube.geometry.parameters.height / 2 + 0.01
    // 添加到场景中去
    scene.add(cube)

    // 摄影机空间Z轴
    camera.position.set(5, 5, 5)
    camera.lookAt(scene.position)


    // 鼠标操作缩放
    // eslint-disable-next-line no-new
    const oribitControls = new OrbitControls(camera, renderer.domElement)
    oribitControls.enableDamping = true


    // 添加帧渲染
    const animate = () => {
      oribitControls.update()
      requestAnimationFrame(animate)
      // 渲染场景
      renderer.render(scene, camera)
    }

    animate()

    // 自适应
    window.addEventListener('resize', () => {
      // 初始化摄像机
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()

      // 初始化渲染器尺寸
      renderer.setSize(window.innerWidth, window.innerHeight)
    })
  }
}
</script>
