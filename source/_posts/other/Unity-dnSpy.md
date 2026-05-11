---
title: Unity 游戏拆包与 C# 逆向初探
date: 2025-04-09
category:
  - 技术
  - 逆向工程
tag:
  - Unity
  - CSharp
  - dnSpy
  - AssetStudio
description: 记录一次从 Unity 游戏资源拆包、DLL 反编译，到 Hook 与内存修改的完整认知过程。
---

# Unity 游戏拆包与 C# 逆向初探

第一次真正接触 Unity 相关内容时，直觉是：

> 这类游戏的结构好像比传统 C++ 游戏更“透明”。

后来才逐渐确认，这种“透明感”确实存在于不少 Unity 项目中，尤其是中小型游戏。

它们通常由几部分组成：

- Unity 引擎本体
- C# 业务逻辑（Mono 或 IL2CPP）
- 资源文件（Assets / AssetBundle）
- 少量本地配置与热更新逻辑

在没有做额外保护的情况下，很多逻辑确实是可见的。

包括：

- 游戏数值逻辑
- UI 行为
- 抽卡/概率计算
- 网络请求封装
- 存档与本地数据结构
- 部分调试/管理接口

甚至偶尔还能看到：

- 接口地址
- 调试日志开关
- 明文密钥或测试逻辑

这些东西第一次看到时，会比较直观地意识到：客户端其实远没有想象中“封闭”。

---

# Unity 游戏的大致结构

一个典型 Unity 构建产物大致如下：

```txt
Game/
├── Game.exe
├── UnityPlayer.dll
├── Game_Data/
│   ├── Managed/
│   │   ├── Assembly-CSharp.dll
│   │   ├── UnityEngine.dll
│   │   └── ...
│   ├── StreamingAssets/
│   ├── Resources/
│   ├── globalgamemanagers
│   └── sharedassets*.assets
```

````

通常关注点集中在：

| 文件                | 作用           |
| ------------------- | -------------- |
| Assembly-CSharp.dll | 游戏核心逻辑   |
| UnityEngine.dll     | 引擎 API       |
| sharedassets        | 资源数据       |
| StreamingAssets     | 原始资源       |
| globalgamemanagers  | Unity 配置数据 |

---

# Mono 与 IL2CPP

Unity 的核心差异基本由这一点决定。

## Mono

较早期或部分项目使用。

特点：

- C# 编译为 IL 中间语言
- DLL 可直接反编译
- 逻辑结构相对完整

在这种模式下，代码可读性较高。

---

## IL2CPP

后续为性能与安全引入的方案：

```txt
IL -> C++ -> Native Binary
```

特点：

- 不再直接暴露 C# DLL
- 编译为原生二进制
- 逆向链路明显更长

常见结构：

```txt
GameAssembly.dll
global-metadata.dat
```

需要借助额外工具恢复结构信息。

---

# dnSpy：C# 反编译调试工具

第一次使用 dnSpy 时的感受比较直接：

> DLL 不再是“黑盒”。

它可以做到：

- 反编译 C# 代码
- 查看完整逻辑结构
- 动态调试运行状态
- 修改方法实现
- 重新保存程序集

在 Mono 环境下，它几乎等同于源码查看器。

---

# 修改 Assembly-CSharp.dll 的直观体验

例如：

```csharp
public int gold = 100;
```

修改为：

```csharp
public int gold = 999999;
```

保存后重新运行程序，数值变化会直接体现。

这种反馈非常直接，也容易理解客户端逻辑的“可控性”。

---

# 常见可修改内容

## 数值类

```csharp
hp = 99999;
attack = 9999;
```

---

## 冷却逻辑

```csharp
skillCD = 0;
```

---

## 概率逻辑

```csharp
rateSSR = 0.8f;
```

---

## 判定逻辑

```csharp
if (enemyHp <= 0)
```

---

# dnSpy 调试能力

除了静态修改，更重要的是动态调试能力：

- 断点调试
- 单步执行
- 变量观察
- 调用栈分析

在体验上更接近传统 IDE 调试器。

---

# 查找关键逻辑的常见方式

## 关键词搜索

```txt
damage
hp
atk
login
```

---

## 生命周期函数入口

Unity 常见入口：

- Start
- Awake
- Update

---

## UI 事件入口

例如按钮回调：

```csharp
OnClickAttack()
```

---

# AssetStudio：资源解析工具

与 dnSpy 侧重逻辑不同，AssetStudio 主要处理资源层。

支持内容包括：

## 图片资源

- Texture2D
- Sprite
- PNG

---

## 音频资源

- WAV
- OGG

---

## 模型与动画

- Mesh
- Animation
- FBX

---

## Unity 特有资源

- AssetBundle
- ScriptableObject
- Prefab

---

# 资源层的一个现实情况

在不少项目中可以看到：

- 未上线内容残留
- 测试 UI
- 废弃角色资源
- 活动素材

这些内容往往只是未被引用，而非完全不存在。

---

# Unity 资源结构的本质

Unity 的资源体系本质是序列化数据：

- 对象结构被序列化
- 运行时反序列化加载

在未加密情况下，结构可解析性较强。

---

# 热更新体系

后期较多项目会引入：

- AssetBundle
- Addressables
- YooAsset
- HybridCLR

结构逐渐变为：

```txt
本地客户端 + 远程资源/CDN
```

---

# 网络请求与资源下载

常见形式：

```txt
https://cdn.xxx.com/asset/xxx.bundle
```

资源通过 HTTP 直接分发。

---

# DLL 热更新结构

部分项目会采用：

```txt
远程下载 DLL -> 动态加载执行
```

特点：

- 逻辑可替换
- 更新灵活
- 调试空间较大

---

# Harmony：C# 方法 Hook

在 .NET 环境中，Hook 相对直接。

示例：

原函数：

```csharp
public int GetGold()
{
    return 100;
}
```

Hook 后：

```csharp
[HarmonyPatch(typeof(Player), "GetGold")]
class Patch
{
    static bool Prefix(ref int __result)
    {
        __result = 999999;
        return false;
    }
}
```

特点：

- 无需修改原 DLL
- 可运行时生效
- 支持方法替换与拦截

---

# Cheat Engine：内存分析工具

典型流程：

## 数值定位

```
1000 -> 变化 -> 950 -> 收敛
```

---

## 修改与锁定

- 修改内存值
- 锁定数值

---

## 指针链

```txt
基址 + 偏移
```

用于解决地址变化问题。

---

# IL2CPP 带来的变化

IL2CPP 模式下：

```txt
GameAssembly.dll
global-metadata.dat
```

结构变为 native 层逻辑。

常见工具链：

- Il2CppDumper
- IDA
- Ghidra
- Frida
- x64dbg

---

# Frida：动态 Hook 工具

具备运行时注入能力，例如：

```javascript
Interceptor.attach(addr, {
  onEnter(args) {
    console.log(args[0]);
  },
});
```

可用于分析运行行为。

---

# 防护机制演进

常见防护方式：

- IL2CPP 编译
- 代码混淆
- 资源加密
- CRC 校验
- 反调试机制
- 虚拟机壳保护（VMProtect）
- 服务端校验

---

# 客户端与服务端边界

一个关键变化是：

## 单机/弱联网

客户端逻辑占主导。

---

## 强联网/现代游戏

核心逻辑逐渐迁移至服务端：

- 数值计算
- 经济系统
- 掉落概率
- 战斗结算

客户端更多承担展示与交互。

---

# 认知层面的变化

回过头看，这一阶段更重要的变化不在“修改能力”，而在理解：

- 程序运行的结构
- DLL 与 IL 的关系
- 内存与逻辑的对应关系
- 客户端可信边界
- 动态修改与执行模型

这些理解会延伸到：

- Web 安全
- 自动化脚本
- 插件系统
- 动态注入
- 热更新架构
- 客户端/服务端设计

````
