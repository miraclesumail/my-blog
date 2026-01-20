---
title: 'js binary operation'
description: 'flex进阶'
pubDate: 'Dec 08 2025'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

##### 1.使用移位的基本操作

```js
   const a = 2 << 3 = 16;  // 向左移动3位
   const b = 16 >> 3 = 2;  // 向右移动3位
```

向左移结果等于 2 \* Math.pow(2,3), 右移16 / Math.pow(2,3)

```js
取某个数的低32位;
// 50亿，二进制大概是 1 00101010 00000101 11110010 00000000
// 它总共有 33 位（超出了 32 位限制）
let largeNum = 5000000000;

// 使用 >>> 0 换成二进制后，只取最后32位
let low32 = largeNum >>> 0;

console.log(low32);
// 输出: 705032704

实际的运算是：5000000000 % (Math.pow(2, 32))

如果是取高位：
const divider = Math.pow(2, 32) = 4294967296
Math.floor(lenInBits / 4294967296)
```

##### 2.使用Uint操作字符串

```js
function addHello(str) {
  const encoder = new TextEncoder();
  const aa = encoder.encode(str);
  const bb = encoder.encode('world');
  const u8 = new Uint8Array(10);
  u8.set(aa);
  u8.set(bb, aa.length); // 第二个是offset

  console.log(String.fromCharCode(...u8));
}

addHello('hello');
// 打印 helloworld
```

下面展示如何将arraybuffer转化为str

```js
function convertToMsg(buffer) {
  const u8 = new Uint8Array(buffer);
  let str = '';

  for (let i = 0; i < u8.byteLength; i++) {
    str += String.fromCharCode(u8[i]);
  }
  return str;
}

const buffer = new TextEncoder().encode('hello').buffer;
console.log(convertToMsg(buffer));
```

uint8的buffer属性是ArrayBuffer

##### 3.16进制和二进制变换

```js
const num16 = 0x619c;
const num2 = 110000110011100; // num16转化为二进制
num2 = 1100001 + 10011100;

// 使用dataview进行set赋值
const buffer = new ArrayBuffer(2); // 16位是2个字节长度
const view = new DataView(buffer);
view.setUint16(0, 0x619c, false); // false表示大端序，默认也是
console.log(buffer); // Uinit8([97, 156]) 就是1100001和10011100转化为10进制后的数
```

既然到这里了，那么看下这个demo

```js
function test() {
  const buffer = new ArrayBuffer(12);
  const view = new DataView(buffer);
  view.setUint16(0, 0x619c, false);
  view.setUint32(2, 1701209960, false); // 32进制设置直接转化成10进制
  view.setUint8(6, 97);
  const u8 = new Uint8Array(buffer);
  u8.set(new TextEncoder().encode('hello'), 7);
  console.log(buffer);
  console.log(String.fromCharCode(...u8));
}
```

##### 4.合并多个buffer

```js
function combineBuffers(buffers) {
  const totalLength = buffers.reduce((sum, buf) => sum + buf.byteLength, 0);
  const combined = new Uint8Array(totalLength);
  let offset = 0;

  for (const buf of buffers) {
    combined.set(new Uinit8(buf), offset);
    offset += buf.byteLength;
  }

  return combined.buffer;
}

// 测试合并buffer的方法
function testCombine() {
  const buffer1 = new ArrayBuffer(5);
  const view1 = new DataView(buffer1);
  for (let i = 0; i < 5; i++) {
    view1.setUint8(i, i + 65);
  }

  const buffer2 = new ArrayBuffer(5);
  const view2 = new DataView(buffer2);
  for (let i = 0; i < 5; i++) {
    view2.setUint8(i, i + 70);
  }

  const combinedBuffer = combineBuffers([buffer1, buffer2]);
  console.log(String.fromCharCode(...new Uint8Array(combinedBuffer)));

  // 打印ABCDEFGHIJ
}
```

##### 5.dataview赋值截取

```js
const buffer1 = new ArrayBuffer(8);
const view = new DataView(buffer1, 3); // offset = 3, 截取(4--8)
view.setUint8(0, 98);
console.log(view.byteLength); // 5(4-8)的长度
console.log(view.getUint8(0)); // 98
```
