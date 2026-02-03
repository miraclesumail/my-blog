---
title: 'three js begin lesson'
description: 'flex进阶'
pubDate: 'Jan 05 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }
</style>

##### 1.创建场景中的主体，一般是各种geometry

<div class="small-code">

```js
import * as Three from 'three';

// 创建一个立方体(1,1,1)
const boxGeometry = new Three.BoxGeometry(1, 1, 1);

// 创建一个平面的圆 radius = 8, 分区segements = 10, startAngel = 0, 圆弧角度360
const circleGeometry = new Three.CircleGeometry(8, 10, 0, Math.PI * 2);

// 创建一个圆柱体 如果radiusTop=0那么就是圆锥了
/**
 * radiusTop = 8
 * radiusBottom = 10
 * height = 10
 * radialSegments = 6
 * heightSegments = 5
 * openEnded = false 空心的圆柱
 */
const cylinderGeometry = new Three.CylinderGeometry(8, 10, 10, 6, 5, false);

// 创建一个球体 参考文档
const sphereGeometry = new THREE.SphereGeometry(15, 32, 16);

// 以上的geometry都继承于BufferGeometry
const vertices = new Float32Array([
 	 -1.0, -1.0,  1.0, // v0
	 1.0, -1.0,  1.0, // v1
	 1.0,  -1.0,  -1.0, // v2
	 1.0,  -1.0,  -1.0, // v2
    -1.0,  -1.0,  -1.0, // v6
	-1.0, -1.0,  1.0, // v0
]);
const bufferGeometry = new Three.BufferGeometry();

// 创建了一个空间的正方形
bufferGeometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));

// 除了上述方式还可以通过静态fromJson方法创建geometry
const json = {
  metadata: {
    version: 4.7,
    type: 'BufferGeometry',
    generator: 'BufferGeometry.toJSON',
  },
  uuid: '6a72ef94-15ec-46ad-9ca4-cf22aa2b102e',
  type: 'BoxGeometry',
  width: 2,
  height: 2,
  depth: 1,
};

const boxGeometry1 = THREE.BoxGeometry.fromJSON(obj);
```

</div>

##### 2.设置geometry的皮肤，一般是各种material

<div class="small-code">

```js
import * as Three from 'three';
import { RoomEnvironment } from 'https://cdn.jsdelivr.net/npm/three@0.182.0/examples/jsm/environments/RoomEnvironment.js';

// 上面的geometry只是确定形状，但是皮肤还没有
// MeshBasicMaterial不需要光源也能显示 不反光，不接受阴影，色彩扁平。渲染速度最快。
const material = new THREE.MeshBasicMaterial({ color: 'red', wireframe: true });

// 这种材料需要环境光照射
const lamberMaterial = new THREE.MeshLamberMaterial({ color: 0x049ef4, emissive: 0x000000 });

// 以下方法也可以使的lamber材质呈现出本身颜色
function enableMeshLamber() {
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  const pmremGenerator = new THREE.PMREMGenerator(renderer);

  const scene = new THREE.Scene();
  scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 1).texture;

  const geometry = new THREE.TorusKnotGeometry(10, 3, 200, 32).toNonIndexed();

  const mesh = new THREE.Mesh(geometry);
  mesh.material = new THREE.MeshLambertMaterial({ color: 0x049ef4 });

  scene.add(mesh);
}

// 现代 3D 引擎（如 Unity, Unreal）的标准。光影表现非常准确，支持粗糙度和金属度。(需要有light才可显示)
// 如果roughness粗糙度为1， 那么pointLight打在上面没效果
const meshStandardMaterial = new THREE.MeshStandardMaterial({ color: 0x049ef4, roughness: 0, metalness: 1 });

// mesh才是真正展示在场景中的实物
const mesh = new THREE.Mesh(boxGeometry, material);
```

</div>

##### 3.创建场景和相机，渲染到屏幕上

<div class="small-code">

```js
// 创建场景
const scene = new THREE.Scene();

scene.add(mesh1, mesh2);

const aspect = window.innerWidth / window.innerHeight;

// 相机z=10 fov = 75 near = 0.1 far = 1000
const camera = new THREE.PerspectiveCamera(75, aspect, 0.1, 1000);
camera.positon.z = 10;

// 创建渲染器 像素比和尺寸
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}

animate();
```

</div>

##### 4.窗口发生变化时，改变aspect

<div class="small-code">

```js
import { OrbitControls } from 'three/examples/jsm/Addons.js';

// 控制旋转操作
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;

// 调整相机宽高比
function updateCameraAspect() {
  const aspect = window.innerWidth / window.innerHeight;
  camera.aspect = aspect;
  camera.updateProjectionMatrix();
}

// 监听窗口大小变化
window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  updateCameraAspect();
});
```

</div>
