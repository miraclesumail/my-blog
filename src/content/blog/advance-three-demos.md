---
title: 'advance-three-demos'
description: 'threejs进阶'
pubDate: 'Jan 09 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }

  p {
    margin: 0 !important;
  }

  li {
     font-size: 16px !important;  /* 强制覆盖默认大小 */
    line-height: 1.4 !important; /* 调整行高，让排版更紧凑 */
    margin-bottom: 8px;
  }
</style>

1. 🤖 [使用自定义loader加载glb](#glb)
2. ⚙️ [自定义texture](#tech)
3. 🔋 [计算点击物体的鼠标位置](#mouse)
4. 🤸 [使用panel调节变量参数](#layout)
5. 🕸️ [配合gsap的动效](#animate)

#### <a name="glb">🤖 使用自定义loader加载glb</a>

<div class="small-code">

```js
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';

// glb需要加载loader gltf不需要
(function initLoader() {
  const gltfLoader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();

  // set dracoLoader path
  dracoLoader.setDecoderPath('/draco/');
  dracoLoader.preload();

  // set dracoLoader to gtlf loader
  loader.setDRACOLoader(dracoLoader);
  window.loader = loader;
})();

function loadGltf(path) {
  loader.load(
    path,
    (gltf) => {
      const group = gltf.scene;
      scene.add(group);
    },
    (event) => {
      // 当前加载了多少
      console.log(event.loaded);
    },
    (err) => {
      console.log(err);
    },
  );
}
```

</div>
去找node_modules/three/examples/jsm/libs/draco/这个目录复制到public下

#### <a name="tech">⚙️ 自定义texture</a>

<div class="small-code">

```js
function createFuTexture(text = '福') {
  const canvas = document.createElement('canvas');
  canvas.width = 512;
  canvas.height = 512;
  const ctx = canvas.getContext('2d');

  // 1. 画红色背景
  ctx.fillStyle = '#D6000F'; // 中国红
  ctx.fillRect(0, 0, 512, 512);

  // 2. 画金色的边框 (可选)
  ctx.strokeStyle = '#FFD700';
  ctx.lineWidth = 20;
  ctx.strokeRect(10, 10, 492, 492);

  // 3. 画福字
  ctx.fillStyle = '#FFD700'; // 金色
  ctx.font = 'bold 300px "KaiTi", "楷体", "STKaiti", serif'; // 使用楷体
  ctx.textAlign = 'center';
  ctx.textBaseAlign = 'middle';
  ctx.fillText(text, 256, 256);

  // 4. 生成纹理
  const texture = new THREE.CanvasTexture(canvas);
  texture.colorSpace = THREE.SRGBColorSpace;
  return texture;
}

const fuMaterial = new THREE.MeshStandardMaterial({
  map: fuTexture, // 贴上福字
  color: 0xffffff, // 保持白色基底，以免贴图变色
  roughness: 0.4,
});

// 材质的面向 right - left - top - bottom - front - back
const materials = [
  plainMaterial, // Right
  plainMaterial, // Left
  plainMaterial, // Top
  plainMaterial, // Bottom
  fuMaterial, // Front (索引 4：正对相机的 Z+ 面)
  fuMaterial, // Back
];

const geometry = new THREE.BoxGeometry(0.8, 1, 0.1);
const box = new THREE.Mesh(geometry, materials);
```

</div>

#### <a name="mouse">⚙️ 计算点击物体的鼠标位置</a>

<div class="small-code">

```js
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
const group = new THREE.Group();
group.add(obj1, obj2, obj3);

function onMouseClick() {
  // A. 计算鼠标在屏幕上的归一化坐标 (-1 到 +1)
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  // B. 更新射线的方向
  raycaster.setFromCamera(mouse, camera);

  // C. 检测射线与 Group 的相交情况
  const intersects = raycaster.intersectObject(group, true);
}

if (intersects.length > 0) {
  // D. 获取被点击的具体部分
  // intersects 数组是按距离排序的，[0] 是最近的那个（也就是你点的那个）
  const clickedObject = intersects[0].object;

  console.log('你点击了:', clickedObject.name, clickedObject.material.name);
  // changeMaterialColor(clickedObject.name);
  window.nodeName = clickedObject.name;
  // window.materialName = clickedObject.name;
  // E. 改变点击物体的颜色（视觉反馈）
  // clickedObject.material.color.set(0x00ff00);
} else {
  console.log('没点中 Group 里的任何东西');
}
```

</div>

#### <a name="layout">🤸 使用pane调节变量参数</a>

<div class="small-code">

```js
import { Pane } from 'tweakpane';

// 创建mesh并添加到场景中
const boxMesh = new THREE.Mesh(boxGeometry, boxMaterial);
scene.add(boxMesh);

const pane = new Pane();
pane.addBinding(boxMesh.position, 'x', { min: 0, max: 5, step: 0.1 });
```

</div>

#### <a name="animate">🕸️ 配合gsap的动效</a>

<div class="small-code">

```js
const scene = new THREE.Scene();
const meshes = [];
scene.traverse((object) => {
  if (object.isMesh) meshes.push(object);
});

const rotations = meshes.map((mesh) => mesh.rotation);

gsap.to(rotations[0], {
  y: `+=${Math.PI * 2}`,
  x: `-=${Math.PI * 2}`,
  duration: 0.2,
  repeat: -1,
  ease: 'none',
});
```

</div>
