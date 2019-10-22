## 照相机

照相机定义三维空间到二维屏幕的投影方式。

### 正交投影 VS 透视投影

* 正交投影对于在三维空间内平行的线，投影到二维空间中也一定是平行的. 不会改变比例
* 透视投影有近大远小的效果.

```js
// 正交投影
// 六个参数分别代表正交投影照相机拍摄到的空间的六个面的位置，这六个面围成一个长方体，我们称其为视景体（Frustum）
THREE.OrthographicCamera(left, right, top, bottom, near, far)

// 需要保证(right - left) 宽 与(top - bottom) 高的比例与Canvas宽度与高度的比例一致
// 为了保证场景中的物体不会因为太近或太远而被照相机忽略，一般near的值设置得较小，far的值设置得较大

//照相机默认是面向z轴负方向
```

```js
//透视投影
THREE.PerspectiveCamera(fov, aspect, near, far)

// fov是视景体竖直方向上的张角
// aspect等于width / height，是照相机水平方向和竖直方向长度的比值，通常设为Canvas的横纵比例。
// near和far分别是照相机到视景体最近、最远的距离，均为正值，且far应大于near。
```

## 物体

在创建物体时，需要传入两个参数，一个是几何形状（Geometry），另一个是材质（Material）

几何形状决定了物体的顶点位置等信息，材质决定了物体的颜色、纹理等信息。

### 几何形状

* 立方体(CubeGeometry)
* 平面(PlaneGeometry)
* 球体(SphereGeometry)
* 圆形(CircleGeometry)
* 圆柱体(CylinderGeometry)
* 正四面体(TetrahedronGeometry)
* 正八面体(OctahedronGeometry)
* 正二十面体(IcosahedronGeometry)
* 圆环面(TorusGeometry)
* 圆环结(TorusKnotGeometry)

* 文字形状(TextGeometry) [字体库](http://typeface.neocracy.org/)

* 自定义形状

### 材质

材质（Material）是独立于物体顶点信息之外的与渲染效果相关的属性.

通过设置材质可以改变物体的颜色、纹理贴图、光照模式等.

#### 基本材质 （BasicMaterial）

#### Lambert材质（MeshLambertMaterial）

符合 `Lambert` 光照模型的材质。Lambert光照模型的主要特点是只考虑漫反射而不考虑镜面反射的效果，因而对于金属、镜子等需要镜面反射效果的物体就不适应，对于其他大部分物体的漫反射效果都是适用的。

##### Phong材质（MeshPhongMaterial）

符合`Phong` 光照模型的材质。Phong模型考虑了镜面反射的效果，因此对于金属、镜面的表现尤为适合.

#### 法向材质

法向材质可以将材质的颜色设置为其法向量的方向.

#### 材质的纹理贴图

## 光与影

### 环境光

环境光是指场景整体的光照效果，是由于场景内若干光源的多次反射形成的亮度一致的效果，通常用来为整个场景指定一个基础亮度。因此，环境光没有明确的光源位置，在各处形成的亮度也是一致的.

```js
// 在设置环境光时，只需要指定光的颜色
var light = new THREE.AmbientLight(0xffffff);
scene.add(light);
```

环境光通常使用白色或者灰色，作为整体光照的基础

### 点光源

点光源是不计光源大小，可以看作一个点发出的光源。点光源照到不同物体表面的亮度是线性递减的，因此，离点光源距离越远的物体会显得越暗

```js
THREE.PointLight(hex, intensity, distance)
```

### 平行光

对于任意平行的平面，平行光照射的亮度都是相同的，而与平面所在位置无关。

```js
THREE.DirectionalLight(hex, intensity)
```

平面亮度与平面的位置无关，而只与平面的法向量相关。只要平面是平行的，那么得到的光照也一定是相同的。

### 聚光灯

聚光灯是一种特殊的点光源，它能够朝着一个方向投射光线。聚光灯投射出的是类似圆锥形的光线，这与我们现实中看到的聚光灯是一致的。

```js
THREE.SpotLight(hex, intensity, distance, angle, exponent)
```

### 阴影

能形成阴影的光源只有THREE.DirectionalLight与THREE.SpotLight；而相对地，能表现阴影效果的材质只有THREE.LambertMaterial与THREE.PhongMaterial

## 着色器

### 顶点着色器

### 片元着色器
