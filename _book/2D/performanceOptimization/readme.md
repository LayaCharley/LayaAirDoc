# 性能优化



## 一、 性能优化

### 1.1  减少动态属性查找

JavaScript中任何对象都是动态的，你可以任意地添加属性。然而，在大量的属性里查找某属性可能很耗时。如果需要频繁使用某个属性值，可以使用局部变量来保存它：

```typescript
foo()
{
    var prop=this.target.prop;
    //使用prop
    this.process1(prop);
    this.process2(prop);
    this.process3(prop);
}
```

### 1.2  性能消耗的回收

日常在使用消耗性能的功能时，尤其是循环处理，当无需使用时，一定要及时回收，或停止循环。



 LayaAir提供两种计时器循环来执行代码块。

1. `Laya.timer.frameLoop`执行频率依赖于帧频率，可通过Stat.FPS查看当前帧频。


1. `Laya.timer.loop`执行频率依赖于参数指定时间。

```typescript
Laya.timer.frameLoop(1, this, this.animateFrameRateBased);
Laya.stage.on("click", this, this.dispose);
dispose() 
{
    Laya.timer.clear(this, this.animateFrameRateBased);
}
```

当一个对象的生命周期结束时，记得清除其内部的Timer：