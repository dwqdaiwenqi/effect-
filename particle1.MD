### Demo:https://tympanus.net/Development/3d-particle-explorations/index3.html#debug



#### 基元
这是一个粒子效果。和大多的粒子效果一样，它都是由一个一个小的“基元”组合而成的，而在本作品中这个基元就是“球体”，也就是说，那一个个粒子本质上是由SphereGeometry加上不同的Material形成的Mesh。对于本例的粒子，主要被设置的显示属性是半径、位置、颜色和透明度。

#### 生长
光有粒子本身是无法实现效果的，我们还需要让它们按照一定的规律动起来。这个动起来包括两部分——其一是粒子自身的约束，其二是粒子间的约束。

首先是粒子自身的约束，这其实就是一个生命周期中，生长和消亡的过程。看到这个效果，我们很自然的可以想到——是不是有一个摄像机在对着粒子群，而粒子是在随着时间向着摄像头运动呢？当然是的，粒子群确实是在进行着“出生 -> 向摄像头运动 -> 消失”的生命循环，但实际上你会发现，仅仅是让粒子改变位置是无法达到这种效果的，我们还必须在粒子的一次生命周期中改变它的大小和透明度，使得变化更加自然
```js
this.mesh.scale.set(scale, scale, scale);
this.mesh.position.setZ(z);
// 4为设定的边界，可以调整
(this.mesh.material as THREE.Material).opacity = (1 - Math.abs(z) / 4) * this.config.opacity;
```
但仅仅这样还是不够的，如果所有粒子的行为都一致，那么实际的效果也只是粒子群同时运动到了某一平面，并没有实际效果的那种层次感。所以必须有一种细致操控每一个粒子的方法，来使得它们的生长具有差异化。这个方法也很简单，就是——给每个粒子设置不同的出生时间born和生命life，然后通过这两个参数控制粒子运动的整个周期，这就可以是的粒子群在宏观上拥有一致的运动曲线和终点，而在微观上每一个粒子又都各有自己的特点：
```js
if (!this.initialized) {
  if (this.current < born) {
    this.current += deltaTime;
    return;
  }

  this.initialized = true;
  this.current = 0;
}

if (this.current + deltaTime > life) {
  return;
}

this.current += deltaTime;
```
这段代码说明了born、life的作用，它们决定了每个粒子开始运动和结束运动的时间。

#### 运动曲线

     -
                          -
                        --
                      --
                   ---
                ---   
            ----
         ---
      ---
    --  
  --       
 -      
-        
