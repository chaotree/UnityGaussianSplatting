# Unity GaussianSplatting API 参考文档

## 概述

本文档提供了 Unity GaussianSplatting Package 的完整 API 参考，包括所有主要类、方法、属性和枚举的详细说明。

## 核心类

### GaussianSplatAsset

**命名空间**: `GaussianSplatting.Runtime`

**描述**: 存储高斯点云数据的 ScriptableObject 资源。

#### 属性

```csharp
// 基本信息
public int formatVersion { get; }           // 格式版本号
public int splatCount { get; }              // splat点数量
public Vector3 boundsMin { get; }           // 边界框最小值
public Vector3 boundsMax { get; }           // 边界框最大值
public Hash128 dataHash { get; }            // 数据哈希值

// 数据格式
public VectorFormat posFormat { get; }      // 位置数据格式
public VectorFormat scaleFormat { get; }    // 缩放数据格式
public ColorFormat colorFormat { get; }     // 颜色数据格式
public SHFormat shFormat { get; }           // 球谐函数数据格式

// 数据资源
public TextAsset posData { get; }           // 位置数据文件
public TextAsset colorData { get; }         // 颜色数据文件
public TextAsset otherData { get; }         // 其他数据文件
public TextAsset shData { get; }            // 球谐函数数据文件
public TextAsset chunkData { get; }         // 分块数据文件
public CameraInfo[] cameras { get; }        // 相机信息数组
```

#### 常量

```csharp
public const int kCurrentVersion = 2023_10_20;  // 当前版本
public const int kChunkSize = 256;              // 分块大小
public const int kTextureWidth = 2048;          // 纹理宽度
public const int kMaxSplats = 8_600_000;        // 最大splat数量
```

#### 枚举

```csharp
// 向量格式
public enum VectorFormat
{
    Float32,    // 32位浮点数 (12字节)
    Norm16,     // 16位归一化 (6字节)
    Norm11,     // 11位归一化 (4字节)
    Norm6       // 6位归一化 (2字节)
}

// 颜色格式
public enum ColorFormat
{
    Float32x4,  // 32位浮点数4通道 (16字节)
    Float16x4,  // 16位浮点数4通道 (8字节)
    Norm8x4,    // 8位归一化4通道 (4字节)
    BC7         // BC7压缩纹理 (1字节)
}

// 球谐函数格式
public enum SHFormat
{
    Float32,    // 32位浮点数
    Float16,    // 16位浮点数
    Norm11,     // 11位归一化
    Norm6,      // 6位归一化
    Cluster64k, // 64K聚类
    Cluster32k, // 32K聚类
    Cluster16k, // 16K聚类
    Cluster8k,  // 8K聚类
    Cluster4k   // 4K聚类
}
```

#### 静态方法

```csharp
// 计算数据大小
public static int GetVectorSize(VectorFormat fmt)
public static int GetColorSize(ColorFormat fmt)
public static int GetSHCount(SHFormat fmt, int splatCount)
public static (int,int) CalcTextureSize(int splatCount)
public static GraphicsFormat ColorFormatToGraphics(ColorFormat format)

// 计算内存大小
public static long CalcPosDataSize(int splatCount, VectorFormat formatPos)
public static long CalcOtherDataSize(int splatCount, VectorFormat formatScale)
public static long CalcColorDataSize(int splatCount, ColorFormat formatColor)
public static long CalcSHDataSize(int splatCount, SHFormat formatSh)
public static long CalcChunkDataSize(int splatCount)
```

### GaussianSplatRenderer

**命名空间**: `GaussianSplatting.Runtime`

**描述**: 主要的渲染组件，负责 GPU 资源管理和渲染调度。

#### 属性

```csharp
// 基本属性
public GaussianSplatAsset asset { get; }           // 关联的资产
public int splatCount { get; }                     // splat数量
public bool HasValidAsset { get; }                 // 是否有有效资产
public bool HasValidRenderSetup { get; }           // 是否有有效渲染设置

// 渲染参数
[SerializeField] public int m_RenderOrder;         // 渲染顺序
[SerializeField] public float m_SplatScale;        // splat缩放
[SerializeField] public float m_MaxDistance;       // 最大距离
[SerializeField] public float m_OpacityScale;      // 透明度缩放
[SerializeField] public int m_SHOrder;             // 球谐函数阶数
[SerializeField] public bool m_SHOnly;             // 仅显示球谐函数
[SerializeField] public RenderMode m_RenderMode;   // 渲染模式
[SerializeField] public float m_PointDisplaySize;  // 点显示大小

// 编辑相关
[field: NonSerialized] public bool editModified { get; private set; }
[field: NonSerialized] public uint editSelectedSplats { get; private set; }
[field: NonSerialized] public uint editDeletedSplats { get; private set; }
[field: NonSerialized] public uint editCutSplats { get; private set; }
[field: NonSerialized] public Bounds editSelectedBounds { get; private set; }
```

#### 枚举

```csharp
public enum RenderMode
{
    Splats,             // 正常渲染
    DebugPoints,        // 调试点
    DebugPointIndices,  // 调试点索引
    DebugBoxes,         // 调试边界框
    DebugChunkBounds    // 调试分块边界
}
```

#### 公共方法

```csharp
// 生命周期
public void OnEnable()
public void OnDisable()
public void Update()

// 渲染相关
public void EnsureMaterials()
public void EnsureSorterAndRegister()
internal void CalcViewData(CommandBuffer cmb, Camera cam)
internal void SortPoints(CommandBuffer cmd, Camera cam, Matrix4x4 matrix)

// 编辑功能
public void EditStoreSelectionMouseDown()
public void EditStorePosMouseDown()
public void EditStoreOtherMouseDown()
public void EditUpdateSelection(Vector2 rectMin, Vector2 rectMax, Camera cam, bool subtract)
public void EditTranslateSelection(Vector3 localSpacePosDelta)
public void EditRotateSelection(Vector3 localSpaceCenter, Matrix4x4 localToWorld, Matrix4x4 worldToLocal, Quaternion rotation)
public void EditScaleSelection(Vector3 localSpaceCenter, Matrix4x4 localToWorld, Matrix4x4 worldToLocal, Vector3 scale)
public void EditDeleteSelected()
public void EditSelectAll()
public void EditDeselectAll()
public void EditInvertSelection()
public bool EditExportData(GraphicsBuffer dstData, bool bakeTransform)
public void EditSetSplatCount(int newSplatCount)
public void EditCopySplatsInto(GaussianSplatRenderer dst, int copySrcStartIndex, int copyDstStartIndex, int copyCount)

// 相机控制
public void ActivateCamera(int index)
public void UpdateEditCountsAndBounds()
```

### GaussianSplatRenderSystem

**命名空间**: `GaussianSplatting.Runtime`

**描述**: 全局渲染系统，管理所有活跃的 GaussianSplatRenderer。

#### 静态属性

```csharp
public static GaussianSplatRenderSystem instance { get; }
```

#### 公共方法

```csharp
// 注册管理
public void RegisterSplat(GaussianSplatRenderer r)
public void UnregisterSplat(GaussianSplatRenderer r)

// 渲染相关
public bool GatherSplatsForCamera(Camera cam)
public Material SortAndRenderSplats(Camera cam, CommandBuffer cmb)
public CommandBuffer InitialClearCmdBuffer(Camera cam)
```

#### 内部字段

```csharp
internal static readonly ProfilerMarker s_ProfDraw;
internal static readonly ProfilerMarker s_ProfCompose;
internal static readonly ProfilerMarker s_ProfCalcView;
```

### GaussianSplatURPFeature

**命名空间**: `GaussianSplatting.Runtime`

**描述**: URP 渲染管线集成特性。

#### 类结构

```csharp
class GaussianSplatURPFeature : ScriptableRendererFeature
{
    class GSRenderPass : ScriptableRenderPass
    {
        // 渲染通道实现
    }
}
```

#### 方法

```csharp
public override void Create()
public override void OnCameraPreCull(ScriptableRenderer renderer, in CameraData cameraData)
public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
protected override void Dispose(bool disposing)
```

### GaussianSplatHDRPPass

**命名空间**: `GaussianSplatting.Runtime`

**描述**: HDRP 渲染管线集成通道。

#### 方法

```csharp
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
```

## 编辑器类

### GaussianSplatAssetCreator

**命名空间**: `GaussianSplatting.Editor`

**描述**: 创建 GaussianSplatAsset 的编辑器窗口。

#### 静态方法

```csharp
[MenuItem("Tools/Gaussian Splats/Create GaussianSplatAsset")]
public static void Init()
```

#### 枚举

```csharp
enum DataQuality
{
    VeryHigh,
    High,
    Medium,
    Low,
    VeryLow,
    Custom
}
```

#### 字段

```csharp
[SerializeField] string m_InputFile;                    // 输入文件路径
[SerializeField] bool m_ImportCameras;                  // 是否导入相机
[SerializeField] string m_OutputFolder;                 // 输出文件夹
[SerializeField] DataQuality m_Quality;                 // 数据质量
[SerializeField] GaussianSplatAsset.VectorFormat m_FormatPos;
[SerializeField] GaussianSplatAsset.VectorFormat m_FormatScale;
[SerializeField] GaussianSplatAsset.ColorFormat m_FormatColor;
[SerializeField] GaussianSplatAsset.SHFormat m_FormatSH;
```

### GaussianSplatRendererEditor

**命名空间**: `GaussianSplatting.Editor`

**描述**: GaussianSplatRenderer 的自定义编辑器。

#### 方法

```csharp
public override void OnInspectorGUI()
public override bool HasPreviewGUI()
public override void OnPreviewGUI(Rect r, GUIStyle background)
```

### GaussianSplatValidator

**命名空间**: `GaussianSplatting.Editor`

**描述**: 验证 GaussianSplatAsset 的有效性。

#### 静态方法

```csharp
public static bool ValidateAsset(GaussianSplatAsset asset, out string errorMessage)
public static bool ValidateRenderer(GaussianSplatRenderer renderer, out string errorMessage)
```

## 工具类

### GaussianUtils

**命名空间**: `GaussianSplatting.Runtime`

**描述**: 高斯渲染工具类。

#### 静态方法

```csharp
public static void FlipProjectionIfBackbuffer(ref Vector4 vertex)
public static float3 LoadVector3(ByteAddressBuffer buffer, uint index, VectorFormat format)
public static float4 LoadVector4(ByteAddressBuffer buffer, uint index, VectorFormat format)
public static float4 LoadQuaternion(ByteAddressBuffer buffer, uint index, VectorFormat format)
```

### GpuSorting

**命名空间**: `GaussianSplatting.Runtime`

**描述**: GPU 排序工具类。

#### 结构体

```csharp
public struct Args
{
    public GraphicsBuffer keys;
    public GraphicsBuffer payload;
    public GraphicsBuffer tempKeys;
    public GraphicsBuffer tempPayload;
    public int count;
    public int keyBits;
    public int payloadBits;
    public bool ascending;
}
```

#### 方法

```csharp
public void Sort(CommandBuffer cmd, Args args)
public void Dispose()
```

## 着色器相关

### 着色器文件

- `GaussianSplatting.hlsl` - 核心着色器库
- `RenderGaussianSplats.shader` - 主渲染着色器
- `GaussianComposite.shader` - 合成着色器
- `SplatUtilities.compute` - 计算着色器
- `DeviceRadixSort.hlsl` - GPU 排序着色器

### 着色器属性

```hlsl
// 数据缓冲区
StructuredBuffer<uint> _OrderBuffer;
StructuredBuffer<SplatViewData> _SplatViewData;
ByteAddressBuffer _SplatSelectedBits;
ByteAddressBuffer _SplatDeletedBits;

// 变换矩阵
float4x4 _MatrixObjectToWorld;
float4x4 _MatrixWorldToObject;
float4x4 _MatrixMV;

// 渲染参数
float _SplatScale;
float _MaxDistance;
float _SplatOpacityScale;
uint _SHOrder;
uint _SHOnly;
float2 _ScreenParams;
float3 _WorldSpaceCameraPos;

// 渲染目标
Texture2D _GaussianSplatRT;
```

## 数据结构

### SplatData

```csharp
struct SplatData
{
    public float3 pos;      // 位置
    public float4 rot;      // 旋转 (四元数)
    public float3 scale;    // 缩放
    public float4 color;    // 颜色
    public float3 sh;       // 球谐函数系数
}
```

### SplatViewData

```csharp
struct SplatViewData
{
    public float4 pos;      // 裁剪空间位置
    public uint2 color;     // 颜色 (半精度)
    public float2 axis1;    // 椭圆轴1
    public float2 axis2;    // 椭圆轴2
}
```

### CameraInfo

```csharp
[Serializable]
public struct CameraInfo
{
    public Vector3 pos;     // 相机位置
    public Vector3 axisX;   // X轴方向
    public Vector3 axisY;   // Y轴方向
    public Vector3 axisZ;   // Z轴方向
    public float fov;       // 视场角
}
```

## 性能分析

### Profiler 标记

```csharp
// 渲染性能标记
internal static readonly ProfilerMarker s_ProfDraw;
internal static readonly ProfilerMarker s_ProfCompose;
internal static readonly ProfilerMarker s_ProfCalcView;
internal static readonly ProfilerMarker s_ProfSort;
```

### 内存管理

```csharp
// 缓冲区管理
static void DisposeBuffer(ref GraphicsBuffer buf)
void DisposeResourcesForAsset()
void ClearGraphicsBuffer(GraphicsBuffer buf)
```

## 扩展性

### 自定义着色器

可以通过修改以下文件来扩展功能：

- `GaussianSplatting.hlsl` - 核心数学函数和数据结构
- `RenderGaussianSplats.shader` - 主渲染管线
- `SplatUtilities.compute` - 计算着色器内核

### 渲染管线集成

支持三种渲染管线集成方式：

1. **Built-in Render Pipeline** - 使用 Camera 事件
2. **Universal Render Pipeline (URP)** - 使用 ScriptableRendererFeature
3. **High Definition Render Pipeline (HDRP)** - 使用自定义渲染通道

### 数据格式扩展

可以通过以下方式扩展数据格式：

1. 添加新的 VectorFormat 枚举值
2. 实现相应的编码/解码函数
3. 更新 GetVectorSize 等静态方法

## 最佳实践

### 性能优化

1. **数据压缩**: 根据质量需求选择合适的压缩格式
2. **分块处理**: 使用 256 个 splat 的分块优化缓存访问
3. **GPU 排序**: 利用 GPU 基数排序提高性能
4. **视锥体剔除**: 只渲染可见的 splat

### 内存管理

1. **及时释放**: 在 OnDisable 中释放 GPU 资源
2. **缓冲区复用**: 尽可能复用 GraphicsBuffer
3. **纹理压缩**: 使用 BC7 等压缩格式减少内存占用

### 渲染质量

1. **透明度排序**: 确保从后到前的正确排序
2. **抗锯齿**: 使用适当的高斯权重计算
3. **光照模型**: 利用球谐函数实现高质量光照

## 故障排除

### 常见问题

1. **内存不足**: 检查 splat 数量是否超过 8.6M 限制
2. **渲染错误**: 验证资产格式版本和完整性
3. **性能问题**: 使用 Profiler 分析瓶颈
4. **兼容性问题**: 确认 Unity 版本和渲染管线支持

### 调试工具

1. **渲染模式**: 使用 Debug 模式查看内部状态
2. **性能分析**: 利用 Unity Profiler 监控性能
3. **数据验证**: 使用 GaussianSplatValidator 验证资产
4. **编辑器工具**: 利用内置编辑器工具进行调试
