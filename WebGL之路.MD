
将学习webgl的过程写下来。

### 一、尝试Three.js。

[Three.js官方文档](https://threejs.org/docs/index.html)  
跟着官方教程写了一遍，初次接触渲染器Renderer，场景Scene，相机Camera，几何体Geometry，材质Material，网格Mesh。

**理解**：  
Scene是个容器，往内添加物体就行。  
Mesh才是真正的物体，Geometry还不完整，需要和Material组合成Mesh。  
Camera也可以当成一个物体设置位置，默认看向z轴的负方向。  

### 二、尝试不同材质和贴图。

**接触光照**：  
从MeshBasicMaterial材质换到MeshPhongMaterial时，物体不显示，因为这个时候需要光照。需不需要光照，是材质决定的。

光照有很多类型，我添加最简单的环境光。

**接触贴图**：  
当前理解，贴图就是贴一张图上去，我往立方体贴了六张图。  
效果：[lesson2](http://test.ttshangxiang.com/three-study/lesson2)

### 三、第一人称视角。

**搭建场景**：  
天空是个球，使用SphereGeometry。  
地面是块板，使用CircleGeometry。  

接触欧拉角Euler，打算使用修改相机欧拉角的方式，调整视野。  
修改相机position，实现走动。

**效果**：[lesson4](http://test.ttshangxiang.com/three-study/lesson4)  
这个效果存在问题，Y轴转动时不正确。

请教前辈后，改用四元数Quaternion调整视野，效果正确。  
**效果**：[lesson5](http://test.ttshangxiang.com/three-study/lesson5) 

后面发现，使用球坐标Spherical配合相机的lookAt，更适用于这个场景。通过控制纬度的大小，可以使画面不会翻转。  

另外还添加了跳跃与射击：  
跳跃通过修改相机的postion的y实现。  
射击通过射线Raycaster实现。  
**效果**：[lesson6](http://test.ttshangxiang.com/three-study/lesson6)  

**学到知识**：欧拉角Euler，四元数Quaternion，球坐标Spherical，射线Raycaster

### 四、点击白板添加文字。

**目标**：文字大小随视角改变。

**过程**：
1. 使用canvas绘制文字，使用CanvasTexture制成纹理。  
2. 接触精灵，使用Sprite加SpriteMaterial创建物体，通过设置spriteMaterial.sizeAttenuation = false，使得精灵大小不变。  
3. 使用Raycaster实现点击添加精灵。

前辈说，他让我做这个的目的不是这个，希望我不用精灵的sizeAttenuation属性实现，并实现设置字体大小功能。  

**过程**：  
1. 通过模拟场景内有一块板，正好覆盖整个屏幕，此时坐标值1对应1像素，计算出相机距离此板的距离d1。
2. 计算出物体距离相机的距离d2。
3. d1 = d2时，1像素正好对应1单位，由于近大远小，则当前物体1像素对应d2 / d1单位。
4. 精灵大小为1单位，我们需要显示fontsize个像素，则需要放大fontsize * d2 / d1。

使用相机的场视角fov与三角函数tan，renderer的高度，计算d1：  
const d1 = height / Math.tan(camera.fov / 2)。  
使用相机位置与物体位置计算d2：  
d2 = camera.postion.distanceTo(sprite.position)。  

结果出现问题，文字越偏离中心，字体越大。  
请教前辈后，得知需要把d2改为两个点的距离到相机Direction的投影，并学习到点乘。
```javascript
const v2 = new THREE.Vector3().subVectors(sprite.position, camera.position)
const d = v2.dot(direction)
```
**效果**：[lesson7](http://test.ttshangxiang.com/three-study/lesson7)

**学到知识**：场视角fov的作用，点乘。

### 五、模仿酷炫文字粒子效果

**步骤**
1. 用canvas画出粒子效果的贴图
2. 使用TextGeometry创建文字几何体。
3. 使用PointsMaterial创建材质，使用粒子贴图。
4. 使用Geometry创建新的几何体，把TextGeometry的顶点复制进来。
5. 用Points结合Geometry与PointsMaterial，创建物体。
5. 实时对Geometry的顶点作某些随机偏移，形成动画效果。

动画算法是复制的：
```javascript
let distance = Math.abs(item.x - o.x) + Math.abs(item.y - o.y) + Math.abs(item.z - o.z);
if (distance > 1){
    item.x += ((o.x - item.x)/distance) * speedX * (1 - Math.random());
    item.y += ((o.y - item.y)/distance) * speedY * (1 - Math.random());
    item.z += ((o.z - item.z)/distance) * speedZ * (1 - Math.random());
}
```
使用这个算法，会得到随机点，然后慢慢收缩回原始点。  
通过改变这个算法，会得到很多不同的效果。

**效果**：[lesson9](http://test.ttshangxiang.com/three-study/lesson9)

**学到知识**：Geometry的顶点的属性是可以随便修改的。

### 六、原生webgl画立方体
此后进入原生WebGL的学习。 

**教程**：[WebGL理论基础](https://webglfundamentals.org/webgl/lessons/zh_cn/)  
这是非常好的教程，我花了很多时间在上面，此处略过很多学习过程。

学习了GLSL的基本语法，与WebGL的基本知识后，我开始画立方体。  
**步骤**：  
1. 构建立方体，添加8个顶点，顶点组合成12个三角形面，以及每个面的法向量。
2. 照例编写简单的shader。
3. 使用一系列方法getAttribLocation、createBuffer、bindBuffer、bufferData、enableVertexAttribArray、vertexAttribPointer创建buffer，传递值等。
4. 设置视图、清空画布、屏蔽反面、深度测试、等准备工作。
5. 使用m4.js提供的透视算法，计算投影矩阵。
6. drawArrays绘制图形。

**效果**：[lesson10](http://test.ttshangxiang.com/three-study/lesson10)

### 七、理解各种矩阵
通过此视频加深了对矩阵的了解：[视频](https://www.youtube.com/watch?v=kjBOesZCoqc&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)

我的理解:  
**矩阵**：3x3的矩阵即x轴，y轴，z轴三个轴组合而成，所以初始矩阵(列向量)为：  
|1, 0, 0|  
|0, 1, 0|  
|0, 0, 1|  
vector3 = [x, y, z]与此矩阵相乘，都只会得到自己的值[x, y, z]，如果修改  
x轴 = v1 = [m11, m12, m13]  
y轴 = v2 = [m21, m22, m23]  
z轴 = v3 = [m31, m32, m33]  
则列矩阵为：  
|m11, m21, m31|  
|m12, m22, m32|  
|m13, m23, m33|  
vector3与此矩阵相乘，可以理解为在v1上的值为x，在v2方向的值为y，在v3方向的值为z。
其实在那个坐标系，他的值仍为[x, y, z]，只是换算到正常坐标轴，值为：  
[xm11 + ym21 + zm31, xm12 + ym22 + zm32, xm13 + ym23 + zm33]  
此时，向量会发生缩放与旋转，不会发生位移。  
人们为了方便，补上位移，扩展成4x4矩阵，加上了平移功能。  
|m11, m21, m31, tx|  
|m12, m22, m32, ty|  
|m13, m23, m33, tz|  
|0,  0,  0,  1|
为了与4x4矩阵相乘，向量需要补上第四位w=1。

**物体矩阵**：物体的缩放，旋转，平移。

**相机矩阵**：相机的位置，朝向组成的矩阵。

**视图矩阵**：相机矩阵的逆矩阵，其实相机移动时，相机并没有移动，而是所有物体朝着相机移动的反方向在移动，所以每个物体都要乘以视图矩阵。

**投影矩阵**：将坐标值的xyz都转换到[-1, 1]范围内的矩阵，因为webgl只接收-1到1的值。  

**正交投影矩阵**：上下左右远近，按比例变换。  

**透视投影矩阵**：场视角，纵横比，远近。近大远小是利用相机坐标系的x与y除以-z的原理，按照这个原理，near，far对应到z的-1，1，再巧妙利用webgl中点xyzw的xyz会自动除以w的原理，能计算出完美的矩阵。  

### 八、原生webgl光照

webgl中，光照并不存在实体，它只是一组影响颜色的向量。

简单光照模型如下：  
**环境光**：光颜色与表面颜色相乘。  
**漫反射光**：简化为光线逆向量(单位化)与法线(单位化)的点乘值，再乘以光颜色、乘以表面颜色。  
**镜面反射光**：点到眼的向量(单位化)与光线逆向量(单位化)相加获得的中线（单位化），与法线(单位化)的点乘值，乘以高光颜色，可通过pow点乘值修改高光范围。  

最终颜色 = 环境光 + 漫反射光 + 镜面反射光。

**效果**：[lesson11](http://test.ttshangxiang.com/three-study/lesson11)

### 九、原生webgl贴图

贴图受贴图坐标影响，在创建几何体时，可以给每个顶点设置uv贴图的二维坐标，从左下角开始，范围是[0, 0]到[1, 1]，意思是此点在这张图的哪个位置。

因此，可以通过调整uv坐标，实现一张图片的各个区域显示在不同的面上，类似于css中用background-postion实现的雪碧图。

**步骤**：
1. 创建gl.createTexture()
2. 激活贴图单元，gl.activeTexture(gl.TEXTURE0)，一共有0-7，八个单元，即一次渲染同时只支持传入8个纹理，默认绑定gl.TEXTURE0，只有一个纹理时不需要写。
3. 绑定纹理，gl.bindTexture(target, texture)，将创建的纹理绑定到激活的单元。
4. 设置采样方法，gl.texParameteri(target, pname, param)，即放大缩小，以及未覆盖完整时的处理办法。
5. 设置纹理内容，gl.texImage2D(...)。
6. 如果缩小采样时用到了mipmap，还要设置mipmap，gl.generateMipmap(gl.TEXTURE_2D)，注意，得等图片加载完后再调用此函数，否则无效。

**效果**：[lesson12](http://test.ttshangxiang.com/three-study/lesson12)

### 十、手动画球体

**步骤**：
1. 南北两极为固定点。
2. 竖向分割，从两极按照设置的段数，均匀画一组点，作为原始弧线。
3. 横向分割，从x负极起，逆时针回到负极，以原始弧线为基本，根据设置的段数，均匀计算一组弧线，此组点即为圆上所有点。
4. 遍历弧线，左右两条弧线间四个点，画两个三角形，两极分别只画一个。
5. 遍历弧线时，顺便计算法向量值，圆上的点，法向量即为坐标点的值，注意单位化。

**效果**：[lesson13](http://test.ttshangxiang.com/three-study/lesson13)

### 十一、阴影

此阴影通过深度纹理创建。  
可以查看此[教程](https://learnopengl.com/Advanced-Lighting/Shadows/Shadow-Mapping)

原理是通过相机模拟，如果眼睛在灯光处，则看得见的点在灯光下，看不见的点在阴影里。
1. 创建阴影着色器，通过在灯光处使用相机，获取可见像素的深度，并输出到纹理保存。
2. 将灯光投影矩阵转换到由[-1, 1]转换到[0, 1]，方便后面计算深度值，并传入正常着色器。
3. 使用灯光投影矩阵转换当前坐标，获取z的值作为深度值a，并使用xy作为纹理坐标，获取阴影纹理对应的值作为深度值b。
4. 对比两个深度值，如果当前点深度值a大于阴影深度值b，则说明此点处于阴影中，画出阴影。

问题：  
**黑色条纹**：阴影贴图只是一张固定宽高的图片，坐标点的分辨率比贴图高，因此会出现多个一块区域的坐标点对应的纹理深度是一样的，这些点的深度有的比它大，有的比它小，大的部分就被当成阴影了。  
**解决办法**：加偏移。  
**偏移优化方法**：bias = max(bias * 10.0 * (1.0 - dot(normal, lightToSurface)), bias); 根据光线与点的法线的点乘，调整偏移。

**阴影位置错误**：因为偏移引起的，发出阴影物体与接受阴影面紧贴时，会发生特别明显的错位。  
**解决办法**：生成阴影纹理时，使用反面作为对象，反面离接受阴影面更近（注意：单面物体不适用）  

**效果**：[lesson14](http://test.ttshangxiang.com/three-study/lesson14)

### 十二、点击白板添加文字2。
使用webgl重写点击白板添加文字，要复杂得多。

**步骤**：  
1. 使用canvas创建纹理等。
2. 创建场景、添加白板。
3. 实现点击，添加文字。
4. 调整文字大小为始终不变。

**难点1**，实现点击，这需要自己写射线。  
思路：  
1. 我们点击时，相当同时点击了相机近点[x, y -1]，与远点[x, y, 1]，得到了两个点，连起来即射线。
2. 将鼠标点击的点pageX，pageY转换到[-1, 1]的范围，能计算出x与y，近远点xy是相同的。
3. 通过乘以逆投影矩阵，逆视图矩阵，可将此两点转到到坐标系中。
4. 网上查到线与面的相交公式：t = (p.n - p0.n) / d.n。p为面是任意点，n为面的法向量，p0为射线起点，d为射线方向，t为射线与面相交时的距离，d.n为0时，平行不相交。
5. 判断相交点是否在面内，此处为长方形，简单使用最大与最小值判断。

**难点2**，文字面对相机，且大小不变。
1. 参考[教程](https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-text-glyphs.html)，可知，视图矩阵只取位移的部分，文字就会面对相机。
2. 文字会随着z值近大远小，提前乘以z值，即可抵消，实现文字不变，除以gl.canvas.height则会使1单位对应1px。

**效果**：[lesson15](http://test.ttshangxiang.com/three-study/lesson15)

### 十三、透明物体

透明使用颜色混合实现，现在开启颜色混合，设置颜色混合方法的参数。
```javascript
gl.enable(gl.BLEND);
gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);
```
混合模式有很多种，这里使用最简单的src_alpha，1 - src_alpha，即物体的透明度颜色加上一减物体透明度的背景颜色。

如果是单个透明物体，通过调整alpha值，已经实现效果了。但是如果是多个透明物体叠加，且后画的在先画的后面，则会显示不正确，解决办法是排序。

将透明物体排序，使用视图矩阵计算z值，用来排序，注意此处z值为负值。

**效果**：[lesson16](http://test.ttshangxiang.com/three-study/lesson16)

### 十四、水池一
偶然看到大神的水池例子，想模仿看看，结果看不太懂，只能按自己的思路来试试看。  
大神例子：<http://madebyevan.com/webgl-water/>

**过程**：  
1. 使用大神的素材，创建了池子与水面。
2. 使用大神素材创建天空盒子纹理。
3. 反射时，假设相机就在与水平面对称的对面，贴天空图，输出到纹理。
4. 折射时，假设相机就往y轴偏移，贴天空图，输出到纹理。
5. 将折射矩阵、反射矩阵、折射贴图、反射贴图输出到水面，通过各个矩阵计算点的位置获取贴图颜色，进行融合。

反射和折射计算了很久，折射还得考虑水上水下不同折射率，水下还有全反射。

折腾这么久，水面还不能动。最后甚至发现，这样写的水面，根本不支持起任何波澜...

**效果**：[lesson17](http://test.ttshangxiang.com/three-study/lesson17)

### 十五、水池二
上面例子，无法模拟水面起波浪，因为通过调整相机视角，折射和反射位置是固定的。

**思路**：怎样才能通过光线位置获得正确的颜色呢？我想到了THREE.CubeCamera，实时生成天空图，然后根据光线与泳池的交点，获取泳池的颜色，高于水池则用天空贴图颜色。

**难点1**，现在不是用Three.js，没有CubeCamera，只能翻开Three.js代码仿制。  
原理：创建六个纹理，六个相机朝向，使用90度fov，每一瞬间都拍摄6副纹理，再组合成TEXTURE_CUBE_MAP。

**难点2**，怎样计算光线与盒子相交的点。  
虽然计算过光线与面相交的点，但这也太复杂了，只好翻大神的代码，发现了一个intersectAABB函数，简单7行，虽然看不懂，但是好用。搭配原本GLSL内置的函数refract、refract（原来还有这种函数），很容易计算出交点。

最后通过三角函数修改了一些顶点的位置与法线，水面开始动了！

**效果**：[lesson18](http://test.ttshangxiang.com/three-study/lesson18)

### 十六、水池三
上面的例子，假如我加入一个球，水面显示球的位置会很奇怪，这样是因为是使用的天空图，只有看最远的物体，才真实。

于是我开始看大神的代码，发现他的水面下的物体，都不是用物体画的，全是靠计算！计算光线先与球还是墙面接触，再计算出颜色。

我抄了过来，放弃了CubeCamera，并添加了点击生成波纹效果。  
**步骤**：
1. 使用之前写的射线，获取点击点的坐标，配合当时的时刻，形成数组。
2. 使用一个shader专门生成纹理贴图，传入上述数组，生成顶点y位移与新法线，保存到rgba四颜色里。
3. 将波纹纹理传入正常水面shader，应用顶点高度与法向量。

波纹碰到墙回弹，应该怎么实现？我想到的方法是，碰到边时，在边的对称位置生成新点，新点与旧点的时刻保持不变，结果一两个点还行，多点几下，生成几千上万个点，uniform无法接收那么多点，而且卡。

**效果**：[lesson19](http://test.ttshangxiang.com/three-study/lesson19)

### 十七、水池四
波浪回弹的问题，我只得看大神是怎么解决的，幸亏慢慢看懂了一些。

我的方式太傻了，大神使用的是仿真的方法，点击则在一处生成一个三角函数的波纹起点，后面则是对比每一点与周围四个点的平均高度，做相应的增减。这样就算点一万次，计算复杂度也不会增加，而且更真实。

我尝试着搬了过来：  
该方法使用两个纹理交替计算。生成波纹、计算波纹、计算法向量，是分开计算的。计算法向量的方法就简单的使用两个向量的叉乘，而且他只用了两个值存储法向量，居然效果挺好。

还需要优化的地方：  
波浪的阴影效果，我始终没看懂，所以自己写了，效果不太好。  
阴影的边界效果很差。  

效果：[lesson20](http://test.ttshangxiang.com/three-study/lesson20)

### 十八、火焰
自此开始使用Three.js的ShaderMaterial。

尝试火焰效果，从shadertoy查到一个比较喜欢的，拿过来用了。  
https://www.shadertoy.com/view/XsXSWS

说实话太多常量参数了，完全不懂逻辑，我只能试着把某些参数修改成能更改的。

**效果**：[lesson21](http://test.ttshangxiang.com/three-study/lesson21)

### 十九、噪声

教程来自：https://www.jianshu.com/p/4c3926c74af0  
跟着教程实现了基本的perlin噪声、worley噪声，以及perlin动态噪声，摸索出了worley噪声的动态版本。

**效果**：[lesson22](http://test.ttshangxiang.com/three-study/lesson22)  
右上角可切换。

### 二十、噪声应用
使用上例生成的ShaderMaterial，配合THREE.WebGLRenderTarget，创建了noise贴图。将它使用到新的ShaderMaterial中，根据明暗值修改物体的顶点值，法向量，得到如下效果。

**效果**：[lesson23](http://test.ttshangxiang.com/three-study/lesson23) 

上面在ShaderMaterial中，需要自己写光照，如果想使用Three.js的光照，则要传入复杂的结构体，而且光线还被视图矩阵换算过...我的水平最多写个phong光照，感觉局限性太大了。

我为什么不生成位移贴图和法向贴图？不用自己写光照了，可以应用各种材质，效果还更好。  
**步骤**：  
1. 使用上例中的噪声着色器，生成噪声贴图。此贴图本来就是黑白的，作为位移贴图刚好合适，乘以一个倍数就行。
2. 使用位移贴图为基础，计算法向量。
3. 将上述两个贴图，传入其他贴图的map里。
4. 正常使用材质和光照绘制物体，此处使用的是MeshPhysicalMaterial。

效果：[lesson24](http://test.ttshangxiang.com/three-study/lesson24) 
