# Gl光照流程中的4种技术
1. 烘焙光照贴图和反射探针
> 通常出现在游戏或低端设备中
2. 自适应探头体积系统（adaptive probe volume system）
> 自动捕捉辐照度的探头网格
3. 光线追踪管道
> 实时光线追踪反射和GL
4. 路径追踪
> 照明质量与易用性的两全策略

# HDPR 基础
## volume
添加覆盖的效果将保存在**配置文件**中  
可有多个配置文件  
可以用它来调整天空、雾、曝光、后期处理等
## HDRI Sky setup
对**间接照明** **漫射照明** **镜面照明**产生巨大影响  
还将决定场景中阴影、阳光、和所有其他人造光之间的照明比例

在没有适当照明的情况下，HDRP将始终回落到HDRI
![常见天空曝光：](../Image/%E5%A4%A9%E7%A9%BA%E5%B8%B8%E8%A7%81%E6%9B%9D%E5%85%89.png "天空常见曝光")
## 太阳和人工照明
volume中添加shadows 使用联级阴影来使阴影之间过度柔和  
volume中添加间接光控制，调整间接漫反射光照，反射光照以增强人工照明
> 如果不使用全局光照，它是一种快速调整不需要的环境照明的方法

# 1. 烘焙光照贴图和反射探针
## 光照贴图
光照贴图是存储光照纹理，运行便宜

lightmap resolution ：25 texels per unit 意味光照贴图每像素4厘米

对于参与GL计算的对象 需要在meshRenderer里打开贡献全局光，收到全局光切换为光照贴图，在这里可以修改光照贴图尺寸

## 反射探头
烘焙后仍会出现部分材质反射天空盒，可通过**反射探针**来修复这种镜面反射泄露  
> rendering debug->lighting->override smooth查看

更多的反射探头会带来更好的效果但意味着更多内存
## 相机曝光
![曝光定义](../Image/%E6%9B%9D%E5%85%89.png)
> rendering debug->lighting->exposure->debugmode:ev100 查看曝光
# 2. 自适应探头体积系统（adaptive probe volume system）
## 传统光照组
### 手动放置  
费时  
对距离物体枢轴最近的四个探测器采样，根据接近度计算光照混合，将混合结果应用于整个物体  

### 自动探针体积  
几乎无漏光，但会有探头存在物体里，需在renderdebug里调整
在volume中添加probe volumes options覆盖调整normal bias，view bias修复因探头密度稀疏产生的阴影问题

# 3. 光线追踪
渲染管道仍然是光栅化技术，但可以用光线追踪对应物替换某些输入
在window->HDRP wizard中选择HDRP+DXR，然后在项目设置中的Quality中选择render pipeline asset并确保光线追踪已开启，还要确保render pipeline asset中的Lights下maxumum lights per cluster足够大以免漏光
## 光线追踪反射
在volume中添加ScreenSpaceReflection覆盖将tracing设置为ray tracing
 - 反射沉闷可调整clamp value但漫射器可能难以处理高值，尤其是粗糙表面上，因为要清理的信号范围要更宽
 - minimum smooth将根据像素平滑度控制哪个像素接收光线追踪反射，较低平滑度将增加渲染成本，其他的采用更便宜的反射探头，甚至可以禁用每种材质的屏幕空间反射以避免任何不必要的成本（在rendering debug 中的lighting下full screen debug mode查看） 
 - 最大光线长度也很重要 

遇到漏光可添加light cluster覆盖调整数量  

为了修复阴影添加ray tracing settings覆盖
- 调整extend shadow culling确保视口外对象进行正确阴影处理
- 调整directional shadow 以修复相机后面潜在的阴影泄露  

## 光线追踪全局照明
因为我们仍然使用烘焙光照贴图，so反射光照并没有更新，因此我们需要光线追踪全局照明  
在volume中添加screen space global illumination覆盖  
- raytracing
- clamp value 增大效果（但会使降噪器工作困难）

# 4. 路径追踪
一键式获得完美照明、阴影、反射  
在volume中添加 pathtracing覆盖  

- 增加最大采样
- 选择降噪器
- 提高intensity

# 总结
![对比图](../Image/%E5%AF%B9%E6%AF%94.png)