---
title: three.js中正反面颜色不同
date: 2023-06-21 10:38:06
tags:
---

## 一、 使用一个材质
利用材质的`OnBeforeCompile`， 例如
```ts
extrudeMeshMat.onBeforeCompile = function(shader) {
  shader.fragmentShader = shader.fragmentShader.replace(
    '#include <dithering_fragment>',
    `
      #ifdef OPAQUE
      diffuseColor.a = 1.0;
      #endif
      
      #ifdef USE_TRANSMISSION
      diffuseColor.a *= material.transmissionAlpha + 0.1;
      #endif
      
      if ( gl_FrontFacing ) { \
        gl_FragColor = vec4( 1.0, 0.0, 0.0, diffuseColor.a ); \
      } else { \
        gl_FragColor = vec4( 0.0, 1.0, 0.0, diffuseColor.a ); \
      }
    `
  )
};
```

着色器参数`diffuseColor`对应的是material.color设置的颜色，gl_FragColor对应的是最终渲染的颜色，gl_FrontFacing表示正面还是反面，在适当的位置，将gl_FragColor的初始化根据gl_FrontFacing对gl_FragColor进行不同的操作就可以得正反面颜色不同的效果

## 二、 使用材质数组
### 1.  创建材质数组，例如
```ts
const material1 = new THREE.MeshBasicMaterial({color: 0xff0000});
const material2 = new THREE.MeshBasicMaterial({color: 0x00ff00});
const materials = [material1, material2]
```

### 2. 如果有顶点索引，则遍历顶点索引，再添加反面的顶点索引；如果没有顶点索引，则遍历顶点，创建正面和反面的顶点索引。使用setIndex函数
```ts
const finalVertices = <THREE.Vector3[]>[...] // 几何体的顶点
const geo = new THREE.BufferGeometry()
geo.setFromPoints(finalVertices)
// ! 计算法向量要在只有一个面的时候计算
geo.computeVertexNormals() 
const newIndexes = <number[]>[]
const count = finalVertices.length
for(let i = 0; i < count; i += 3) {
    newIndexes.push(i, i + 1, i + 2)
}
for(let i = 0; i < count; i += 3) {
    newIndexes.push(i, i + 2, i + 1)
}
geo.setIndex(newIndexes)
```

### 3. 使用setGroup函数，将当前几何体分割成组进行渲染
```ts
geo.addGroup(0, count, 0)
geo.addGroup(count, count, 1)
```

![demo](https://raw.githubusercontent.com/dieice2030/imagesRepo/master/doubleside%20mat2.gif)