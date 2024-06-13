---
title: opencascade.js中拓扑数据的遍历
date: 2024-01-30 16:29:53
tags: 
  - opencascade.js
  - occt
  - 前端
  - 3D
---

## 基本概念
在OCCT中，主要有两种数据，一种是`Geometry`几何数据，一种是`Topology`拓扑数据。

`Topology`拓扑数据是抽象类型可以总结为以下几种:
- `Vertex` 点 0维
- `Edge` 边
- `Wire` 由顶点相连的边组成的序列
- `Face` `plane2D`平面 或者 `surface` 3D表(曲)面 上由`Wire`所围成的一块区域
- `Shell` 由一组通过边连接的面组成的`Wire boundaries`线框
- `Solid` 由`Shell`包围的3D空间的一部分
- `Compound Solid` `Solid`的集合

`Wire` 和 `Solid` 可以闭合也可以`infinity`不闭合

对应到OCCT中的类型有，即`TopAbs_ShapeEnum`的枚举值有:
- `COMPOUND`: 一组任意类型的拓扑数据
- `COMPSOLID`: `Compound Solid`
- `SOLID`: `Solid`
- `SHELL`: `Shell`
- `FACE`: `Face`
- `WIRE`: `Wire`
- `EDGE` `Edge`
- `VERTEX`: `Vertex`
- `SHPAE`: `TopoDS_Shape`是所有类型的基类

## 遍历方法
通常的遍历方法是通过`TopExp_Explorer`  
例如遍历一个面的所有边
```ts
  // oc 是已经加载的 OpenCascadeInstance 实例
  // face 是 TopoDS_Face 类

  // explorer 是一个迭代器
  const explorer = new oc.TopExp_Explorer_2(face, oc.TopAbs_ShapeEnum.TopAbs_EDGE, oc.TopAbs_ShapeEnum.TopAbs_SHAPE)
  while (explorer.More()) {
    // 通过explorer.Current() 拿到当前只向的TopoDS_Shape
    // 由于OCCT基于C++编写，是强类型的，所以需要通过下面的方法声明为TopoDS_Edge类型
    const edge = oc.TopoDS.Edge_1(explorer.Current())
    // 后续操作
    // ...
    // 后续操作
  }

```
不过这样的遍历并不保证顺序，即遍历得到的边不保证是首尾相连的  
如果需要按照顺序遍历
```ts
  // oc 是已经加载的 OpenCascadeInstance 实例
  // face 是 TopoDS_Face 类

  // explorer 是一个迭代器
  // 使用 BRepTools_WireExplorer 替代 TopExp_Explorer 遍历Face/Wire的Edge可以保证顺序
  const explorer = new OcctInstance.oc.BRepTools_WireExplorer_3(outerWire, face)
  while (explorer.More()) {
    // 通过explorer.Current() 拿到当前只向的TopoDS_Shape
    // 由于OCCT基于C++编写，是强类型的，所以需要通过下面的方法声明为TopoDS_Edge类型
    const edge = oc.TopoDS.Edge_1(explorer.Current())
    // 后续操作
    // ...
    // 后续操作
  }

```

## 最佳实践
在`opencascade.js`中，创建的`OCCT`对象不会被`JavaScript`的垃圾回收机制自动清理，需要手动调用`delete`方法删除内存占用。因此，应该尽可能少地创建对象
例如，需要遍历face1 和 face2 的边，可以这样
```ts
  // 在某处创建迭代器
  const explorer = new oc.TopExp_Explorer_1()
  for (explorer.Init(face1, oc.TopAbs_ShapeEnum.TopAbs_EDGE, oc.TopAbs_ShapeEnum.TopAbs_SHAPE); explorer.More(); explorer.Next()) {
    // ...
  }

  for (explorer.Init(face2, oc.TopAbs_ShapeEnum.TopAbs_EDGE, oc.TopAbs_ShapeEnum.TopAbs_SHAPE); explorer.More(); explorer.Next()) {
    // ...
  }

  // 销毁迭代器
  explorer.delete()

```
