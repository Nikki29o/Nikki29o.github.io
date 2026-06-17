---
title: "UE 材质蓝图复杂度评测：从「贴图直连」到「材质体系设计」"
date: 2026-06-17T17:50:00+08:00
draft: false
description: "以 Technical Artist 视角，评测 AI agent 设计复杂 UE 材质体系的能力。在「贴图直连」基线之上，构建 5 类父材质 / 23 个子实例，用四维量化体系打分：套件平均 91.2，相对基线 20.0 实现 4.6× 跃迁。本文完整还原评测报告的数据、配色与图表。"
tags: ["Unreal Engine", "Material", "Shader", "Technical Art", "AI Evaluation", "PBR"]
categories: ["Tool Review"]
---

如果让一个 AI agent 做 UE 材质，它能走多远？最低的标准是「把三张贴图（BaseColor / Normal / Roughness）连到材质输出」——这能还原一个扫描材质，但**那只是资产搬运，不是材质设计**。这篇文章是我以 Technical Artist 的视角做的一次系统评测：在「贴图直连」这条基线之上，看 agent 能否真正**设计出一套复杂材质体系**——参数化、程序化、自定义光照、动态效果、高级 BRDF，并用一套四维量化体系给它打分。

<!--more-->

> 本文的数据、配色与图表均来自配套的交互式评测看板，这里将其适配为博客排版完整呈现。

<style>
.ume{--bg:#0f1115;--panel:#171a21;--panel2:#1e222b;--border:#2a2f3a;--text:#e6e9ef;--muted:#9aa3b2;--good:#3ddc84;--warn:#ffb84d;--bad:#ff5d5d;background:var(--bg);color:var(--text);border:1px solid var(--border);border-radius:14px;padding:22px;margin:22px 0;font-family:"Segoe UI","PingFang SC","Microsoft YaHei",sans-serif}
.ume *{box-sizing:border-box}
.ume h3{color:#fff;border:0;margin:6px 0 14px;font-size:17px}
.ume .kpis{display:grid;grid-template-columns:repeat(5,1fr);gap:10px;margin:6px 0 4px}
.ume .kpi{background:var(--panel2);border:1px solid var(--border);border-radius:10px;padding:13px}
.ume .kpi .v{font-size:23px;font-weight:700;color:#fff}
.ume .kpi .l{font-size:11px;color:var(--muted);margin-top:3px}
.ume .kpi .d{font-size:10px;color:var(--good);margin-top:5px}
.ume .tier{display:flex;gap:0;border-radius:9px;overflow:hidden;margin:6px 0}
.ume .tier>div{flex:1;padding:9px 6px;text-align:center;font-size:11px;color:#fff;line-height:1.4}
.ume table{width:100%;border-collapse:collapse;font-size:13px;margin:0;background:transparent}
.ume th,.ume td{padding:8px 9px;text-align:left;border-bottom:1px solid var(--border);color:var(--text)}
.ume th{color:var(--muted);font-size:12px;font-weight:600;background:transparent}
.ume td.n{text-align:right;font-variant-numeric:tabular-nums}
.ume .bar{height:7px;border-radius:4px;background:var(--panel2);overflow:hidden;min-width:54px}
.ume .bar>i{display:block;height:100%;border-radius:4px}
.ume .g{display:inline-block;width:23px;height:23px;line-height:23px;text-align:center;border-radius:6px;font-weight:700;font-size:12px}
.ume .gS{background:#2e7d52;color:#9dffce}.ume .gA{background:#2a5a9e;color:#bcd8ff}.ume .gD{background:#7a2a2a;color:#ffb0b0}
.ume .cards{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:12px}
.ume .card{background:var(--panel2);border:1px solid var(--border);border-radius:11px;overflow:hidden}
.ume .card .h{padding:12px 13px 9px;border-top:3px solid}
.ume .card .cat{font-size:10px;color:var(--muted);letter-spacing:.5px;text-transform:uppercase}
.ume .card .nm{font-size:15px;font-weight:700;color:#fff;margin:3px 0}
.ume .card .tag{display:inline-block;font-size:10px;padding:2px 8px;border-radius:9px;background:var(--bg);color:var(--muted);margin-top:3px}
.ume .card .tt{float:right;font-size:21px;font-weight:700}
.ume .card .b{padding:0 13px 12px;font-size:12px;color:var(--muted)}
.ume .card .st{display:flex;justify-content:space-between;padding:4px 0;border-top:1px solid var(--border)}
.ume .legend{display:flex;gap:14px;flex-wrap:wrap;font-size:12px;color:var(--muted);margin-top:8px;justify-content:center}
.ume .legend span{display:inline-flex;align-items:center;gap:6px}
.ume .dot{width:10px;height:10px;border-radius:50%;display:inline-block}
.ume .gallery{display:grid;grid-template-columns:repeat(auto-fill,minmax(120px,1fr));gap:10px}
.ume .gi{background:var(--panel2);border:1px solid var(--border);border-radius:9px;overflow:hidden}
.ume .gi img{width:100%;display:block;aspect-ratio:1;object-fit:cover;background:#0a0b0e;margin:0}
.ume .gi .c{padding:6px 8px}
.ume .gi .t{font-size:11px;font-weight:600;color:#fff}
.ume .gi .s{font-size:9px;margin-top:2px}
.ume .bd{display:inline-block;font-size:9px;padding:1px 6px;border-radius:8px;margin-left:4px}
.ume .bt{background:#27405e;color:#a9cdf5}.ume .bb{background:#5e4d27;color:#f5dca9}.ume .be{background:#5e2740;color:#f5a9cd}
.ume .gw{overflow-x:auto;background:#14171d;border:1px solid var(--border);border-radius:11px;padding:10px}
.ume .gw svg{height:auto;margin:0}
.ume .glg{display:flex;gap:13px;flex-wrap:wrap;font-size:11px;color:var(--muted);margin-top:9px}
.ume .glg span{display:inline-flex;align-items:center;gap:5px}
.ume .sq{width:10px;height:10px;border-radius:3px;display:inline-block}
.ume .vs{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-top:6px}
.ume .vs .col{background:var(--panel2);border:1px solid var(--border);border-radius:11px;overflow:hidden}
.ume .vs .col>img{width:100%;display:block;aspect-ratio:1;object-fit:cover;background:#0a0b0e;margin:0}
.ume .vs .ch{padding:10px 12px 4px}
.ume .vs .ch .nm{font-size:14px;font-weight:700;color:#fff}
.ume .vs .ch .sm{font-size:11px;color:var(--muted);margin-top:2px}
.ume .vs .pl{padding:2px 12px 12px;font-size:11.5px}
.ume .vs .pl .r{display:flex;justify-content:space-between;gap:8px;padding:4px 0;border-top:1px solid var(--border)}
.ume .vs .pl .r span{color:var(--muted)}
.ume .vs .pl .r b{color:#fff;font-variant-numeric:tabular-nums}
.ume .sw{display:inline-block;width:11px;height:11px;border-radius:3px;vertical-align:-1px;margin-right:3px;border:1px solid #3a3f4a}
.ume .delta{background:#1a1d24;border-left:3px solid var(--warn);border-radius:0 8px 8px 0;padding:9px 12px;margin-top:12px;font-size:12.5px;color:var(--muted)}
.ume .delta b{color:var(--text)}
@media(max-width:760px){.ume .kpis{grid-template-columns:repeat(2,1fr)}.ume .tier{flex-wrap:wrap}.ume .vs{grid-template-columns:1fr 1fr}}
</style>

## 评测立意：一条基线，五级阶梯

「贴图直连」之所以是基线（T0），是因为它**节点数 ≈ 3、逻辑节点 0、暴露参数 0、可派生实例 1**——没有任何"设计"成分。要衡量 agent 的设计能力，我设计了一条**复杂度阶梯**，每一级对应一类 shader 能力的跃迁：

<div class="ume">
<div class="tier">
<div style="background:#6b7280"><b>T0</b><br>基线·贴图直连</div>
<div style="background:#5b9dff"><b>T1</b><br>参数化</div>
<div style="background:#3ddc84"><b>T2</b><br>程序化</div>
<div style="background:#ffb84d"><b>T3</b><br>动态/风格化</div>
<div style="background:#8b7cff"><b>T4</b><br>专家·高级光照</div>
</div>
</div>

围绕这条阶梯，我让 agent 设计了 **5 类父材质（Master Material）+ 23 个子实例（Material Instance）**，并把每个材质的节点图、暴露参数、连接关系全部结构化为 JSON，再用评分引擎自动量化。下面是总体规模：

<div class="ume">
<div class="kpis">
<div class="kpi"><div class="v">5</div><div class="l">父材质分类</div><div class="d">vs 基线 1 类</div></div>
<div class="kpi"><div class="v">23</div><div class="l">子实例</div><div class="d">vs 基线 1 个</div></div>
<div class="kpi"><div class="v">151</div><div class="l">节点总数</div><div class="d">平均 30 / 材质</div></div>
<div class="kpi"><div class="v">62</div><div class="l">暴露参数</div><div class="d">基线 0 个</div></div>
<div class="kpi"><div class="v">91.2</div><div class="l">套件平均总分</div><div class="d">基线 20.0 分</div></div>
</div>
</div>

## 五类父材质，对应五种 shader 范式

<div class="ume">
<div class="cards">
<div class="card"><div class="h" style="border-color:#5b9dff"><div class="cat" style="color:#5b9dff">PBR 基础类</div><div class="nm">参数化母材质<span class="tt" style="color:#3ddc84">89.8</span></div><span class="tag">T1 参数化</span> <span class="g gA">A</span></div><div class="b"><div class="st"><span>节点 / 深度</span><b style="color:#fff">25 / 8</b></div><div class="st"><span>暴露参数</span><b style="color:#fff">11</b></div><div class="st"><span>着色模型</span><b style="color:#fff">DefaultLit</b></div></div></div>
<div class="card"><div class="h" style="border-color:#3ddc84"><div class="cat" style="color:#3ddc84">程序化纹理类</div><div class="nm">数学节点生成<span class="tt" style="color:#3ddc84">94.3</span></div><span class="tag">T2 程序化</span> <span class="g gS">S</span></div><div class="b"><div class="st"><span>节点 / 深度</span><b style="color:#fff">33 / 11</b></div><div class="st"><span>暴露参数</span><b style="color:#fff">12</b></div><div class="st"><span>着色模型</span><b style="color:#fff">DefaultLit</b></div></div></div>
<div class="card"><div class="h" style="border-color:#ff7eb6"><div class="cat" style="color:#ff7eb6">风格化渲染类</div><div class="nm">卡通色阶+描边<span class="tt" style="color:#ffb84d">86.8</span></div><span class="tag">T3 风格化</span> <span class="g gA">A</span></div><div class="b"><div class="st"><span>节点 / 深度</span><b style="color:#fff">27 / 9</b></div><div class="st"><span>暴露参数</span><b style="color:#fff">11</b></div><div class="st"><span>着色模型</span><b style="color:#fff">Unlit</b></div></div></div>
<div class="card"><div class="h" style="border-color:#ffb84d"><div class="cat" style="color:#ffb84d">动态效果类</div><div class="nm">时间驱动动效<span class="tt" style="color:#3ddc84">96.1</span></div><span class="tag">T3 动态</span> <span class="g gS">S</span></div><div class="b"><div class="st"><span>节点 / 深度</span><b style="color:#fff">37 / 10</b></div><div class="st"><span>暴露参数</span><b style="color:#fff">12</b></div><div class="st"><span>混合模式</span><b style="color:#fff">Masked</b></div></div></div>
<div class="card"><div class="h" style="border-color:#8b7cff"><div class="cat" style="color:#8b7cff">高级光照类</div><div class="nm">多 BRDF 封装<span class="tt" style="color:#3ddc84">88.8</span></div><span class="tag">T4 专家</span> <span class="g gA">A</span></div><div class="b"><div class="st"><span>节点 / 深度</span><b style="color:#fff">29 / 9</b></div><div class="st"><span>暴露参数</span><b style="color:#fff">16</b></div><div class="st"><span>着色模型</span><b style="color:#fff">ClearCoat</b></div></div></div>
</div>
</div>

每类各自考察一种核心能力：

- **PBR 基础类（T1）** — 金属-粗糙度工作流的完整参数化。亮点是 `Roughness Min/Max` 重映射（把贴图灰度 Lerp 到任意粗糙度区间）、Detail Normal 二次混合、AO 强度可调——这些正是 TA 做材质实例化的基本功。
- **程序化纹理类（T2）** — **零贴图**，纯数学生成。多倍频 FBM + Voronoi 细胞噪声混合，叠加 Sine 程序条纹，法线由图案高度的 DDX/DDY 偏导（CustomExpression）程序生成，并用 StaticSwitch 在编译期切换 UV / 世界坐标投影。
- **风格化渲染类（T3 / Unlit）** — **绕过 UE 默认 BRDF** 手动重建光照：手算 N·L → 多档 Posterize 量化成 cel-shading 硬边 → Fresnel 驱动边缘光与轮廓描边。
- **动态效果类（T3 / Masked）** — Time 节点驱动的多合一动效：溶解（切 OpacityMask + 发光边缘）、UV 扭曲、脉冲呼吸、视差、顶点波动（WPO）。
- **高级光照类（T4 / ClearCoat）** — 统一封装四种进阶 BRDF：次表面散射、各向异性（拉丝）、虹彩薄膜干涉、布料绒毛，考察对多种 ShadingModel **专有输出引脚**的掌握。

## 父材质节点蓝图设计图

下面是五类父材质的**节点蓝图**——这是评测里真正被解析、打分的图结构。布局规则统一：**按层级深度从左到右**，最左是输入端（贴图采样、UV、Time、常量、参数），数据流向右经过运算节点，最右一列是连到主材质的**输出引脚**。标有 **★** 的节点即**暴露给材质实例的可调参数**（子材质只覆写这些）。

<div class="ume">
<div class="glg" style="margin-bottom:10px">
<span><span class="sq" style="background:#8b7cff"></span>★ 暴露参数 Parameter</span>
<span><span class="sq" style="background:#5b9dff"></span>贴图采样 Texture</span>
<span><span class="sq" style="background:#3ddc84"></span>噪声 Noise/Voronoi</span>
<span><span class="sq" style="background:#ffb84d"></span>时间/动画 Time/Panner</span>
<span><span class="sq" style="background:#ff7eb6"></span>Custom HLSL</span>
<span><span class="sq" style="background:#ff9d5c"></span>逻辑/数学 If/Switch/Math</span>
<span><span class="sq" style="background:#5a6270"></span>常规运算</span>
</div>
</div>

**① PBR 基础类 `M_PBR_Base_Master`** — 25 节点 / 11 暴露参数。三条管线并行：BaseColor（染色×去饱和）、Normal（强度 + Detail 混合）、Surface（MRA 分离 + 粗糙度 Min/Max 重映射 + AO 强度），汇入 7 个输出引脚。

<div class="ume"><div class="gw"><img src="graphs/M_PBR_Base_Master.svg" alt="PBR 基础类节点蓝图" style="max-width:none;width:1962px"></div></div>

**② 程序化纹理类 `M_Procedural_Master`** — 33 节点 / 12 暴露参数，深度达 11（全套件最深）。注意它**没有任何贴图采样节点**：FBM 与 Voronoi 噪声 + Sine 条纹 → 对比度整形 → 双色渐变；法线由 CustomExpression 对图案高度求偏导生成。

<div class="ume"><div class="gw"><img src="graphs/M_Procedural_Master.svg" alt="程序化纹理类节点蓝图" style="max-width:none;width:2550px"></div></div>

**③ 风格化渲染类 `M_Stylized_Master`** — 27 节点 / 11 暴露参数，Unlit。手算 N·L → Floor/Divide 量化色阶 → 双色带 Lerp；两路 Fresnel 分别驱动边缘光与轮廓描边，最终全部汇入 EmissiveColor 单一输出。

<div class="ume"><div class="gw"><img src="graphs/M_Stylized_Master.svg" alt="风格化渲染类节点蓝图" style="max-width:none;width:2158px"></div></div>

**④ 动态效果类 `M_DynamicFX_Master`** — 36 节点 / 12 暴露参数。Time 节点扇出驱动四套动效（扭曲 / 溶解 / 脉冲 / 视差）+ WPO 顶点波动，输出含 OpacityMask 与 WorldPositionOffset。

<div class="ume"><div class="gw"><img src="graphs/M_DynamicFX_Master.svg" alt="动态效果类节点蓝图" style="max-width:none;width:2354px"></div></div>

**⑤ 高级光照类 `M_AdvLighting_Master`** — 29 节点 / 16 暴露参数（最多），ClearCoat。SSS / 各向异性 / 虹彩 / 布料四组特性并联，分别连到 SubsurfaceColor、Anisotropy+Tangent、EmissiveColor（经 StaticSwitch 裁剪）、Fuzz/Cloth 等专有引脚。

<div class="ume"><div class="gw"><img src="graphs/M_AdvLighting_Master.svg" alt="高级光照类节点蓝图" style="max-width:none;width:2158px"></div></div>

## 子实例生成：典型 / 边界 / 极端三层采样

每个父材质衍生 4–5 个子实例，**仅覆写暴露参数**，不动图结构（这正对应 UE 的 MaterialInstance 机制）。参数选取遵循三层采样，既验证常见场景，也把参数推到边界与极端，探出材质的"表达带宽"：

<div class="ume">
<table>
<tr><th>实例</th><th>采样</th><th>关键覆写</th><th>模拟材质</th></tr>
<tr><td><b>抛光黄铜</b></td><td><span class="bd bt">典型</span></td><td>Metallic=1.0, Roughness 0.05~0.18</td><td>高反射金属</td></tr>
<tr><td><b>做旧锈铁</b></td><td><span class="bd bb">边界</span></td><td>Metallic=0.6, Roughness Max=1.0</td><td>半金属做旧</td></tr>
<tr><td><b>发光面板</b></td><td><span class="bd be">极端</span></td><td>Emissive Intensity=50</td><td>自发光过曝</td></tr>
<tr><td><b>翡翠玉石</b></td><td><span class="bd be">极端</span></td><td>SSS Amount=1.0</td><td>次表面散射</td></tr>
<tr><td><b>拉丝钛金属</b></td><td><span class="bd be">极端</span></td><td>Anisotropy=0.9</td><td>各向异性高光</td></tr>
<tr><td><b>肥皂泡虹彩</b></td><td><span class="bd be">极端</span></td><td>Iridescence Bands=6</td><td>薄膜干涉</td></tr>
<tr><td><b>心跳核心</b></td><td><span class="bd be">极端</span></td><td>Pulse Speed=12, Max=10</td><td>动态脉冲</td></tr>
</table>
</div>

## 23 个子实例的渲染结果索引

下面是全部 23 个子实例的代表性外观预览（依各实例参数语义程序化合成的近似渲染，用于在评测中快速比对视觉差异）。颜色标签对应五类父材质：

<div class="ume">
<div class="gallery">
<div class="gi"><img src="img/r_pbr_brass.png"><div class="c"><div class="t">抛光黄铜<span class="bd bt">典型</span></div><div class="s" style="color:#5b9dff">PBR 基础</div></div></div>
<div class="gi"><img src="img/r_pbr_plastic.png"><div class="c"><div class="t">磨砂塑料<span class="bd bt">典型</span></div><div class="s" style="color:#5b9dff">PBR 基础</div></div></div>
<div class="gi"><img src="img/r_pbr_iron.png"><div class="c"><div class="t">做旧锈铁<span class="bd bb">边界</span></div><div class="s" style="color:#5b9dff">PBR 基础</div></div></div>
<div class="gi"><img src="img/r_pbr_emissive.png"><div class="c"><div class="t">发光面板<span class="bd be">极端</span></div><div class="s" style="color:#5b9dff">PBR 基础</div></div></div>
<div class="gi"><img src="img/r_pbr_default.png"><div class="c"><div class="t">全默认灰<span class="bd bb">边界</span></div><div class="s" style="color:#5b9dff">PBR 基础</div></div></div>
<div class="gi"><img src="img/r_proc_marble.png"><div class="c"><div class="t">程序化大理石<span class="bd bt">典型</span></div><div class="s" style="color:#3ddc84">程序化</div></div></div>
<div class="gi"><img src="img/r_proc_clay.png"><div class="c"><div class="t">龟裂泥土<span class="bd bt">典型</span></div><div class="s" style="color:#3ddc84">程序化</div></div></div>
<div class="gi"><img src="img/r_proc_wood.png"><div class="c"><div class="t">程序化木纹<span class="bd bb">边界</span></div><div class="s" style="color:#3ddc84">程序化</div></div></div>
<div class="gi"><img src="img/r_proc_rock.png"><div class="c"><div class="t">世界投影岩石<span class="bd be">极端</span></div><div class="s" style="color:#3ddc84">程序化</div></div></div>
<div class="gi"><img src="img/r_sty_toon.png"><div class="c"><div class="t">经典二档卡通<span class="bd bt">典型</span></div><div class="s" style="color:#ff7eb6">风格化</div></div></div>
<div class="gi"><img src="img/r_sty_cel.png"><div class="c"><div class="t">多档赛璐珞<span class="bd bt">典型</span></div><div class="s" style="color:#ff7eb6">风格化</div></div></div>
<div class="gi"><img src="img/r_sty_rim.png"><div class="c"><div class="t">强边缘光英雄<span class="bd be">极端</span></div><div class="s" style="color:#ff7eb6">风格化</div></div></div>
<div class="gi"><img src="img/r_sty_ink.png"><div class="c"><div class="t">线稿描边<span class="bd be">极端</span></div><div class="s" style="color:#ff7eb6">风格化</div></div></div>
<div class="gi"><img src="img/r_fx_burn.png"><div class="c"><div class="t">燃烧溶解<span class="bd bt">典型</span></div><div class="s" style="color:#ffb84d">动态效果</div></div></div>
<div class="gi"><img src="img/r_fx_shield.png"><div class="c"><div class="t">能量护盾<span class="bd bt">典型</span></div><div class="s" style="color:#ffb84d">动态效果</div></div></div>
<div class="gi"><img src="img/r_fx_holo.png"><div class="c"><div class="t">全息扰动<span class="bd bb">边界</span></div><div class="s" style="color:#ffb84d">动态效果</div></div></div>
<div class="gi"><img src="img/r_fx_heart.png"><div class="c"><div class="t">心跳核心<span class="bd be">极端</span></div><div class="s" style="color:#ffb84d">动态效果</div></div></div>
<div class="gi"><img src="img/r_fx_water.png"><div class="c"><div class="t">波动水面<span class="bd be">极端</span></div><div class="s" style="color:#ffb84d">动态效果</div></div></div>
<div class="gi"><img src="img/r_adv_jade.png"><div class="c"><div class="t">翡翠玉石<span class="bd be">极端</span></div><div class="s" style="color:#8b7cff">高级光照</div></div></div>
<div class="gi"><img src="img/r_adv_titanium.png"><div class="c"><div class="t">拉丝钛金属<span class="bd be">极端</span></div><div class="s" style="color:#8b7cff">高级光照</div></div></div>
<div class="gi"><img src="img/r_adv_bubble.png"><div class="c"><div class="t">肥皂泡虹彩<span class="bd be">极端</span></div><div class="s" style="color:#8b7cff">高级光照</div></div></div>
<div class="gi"><img src="img/r_adv_carpaint.png"><div class="c"><div class="t">汽车金属漆<span class="bd bt">典型</span></div><div class="s" style="color:#8b7cff">高级光照</div></div></div>
<div class="gi"><img src="img/r_adv_velvet.png"><div class="c"><div class="t">丝绒布料<span class="bd be">极端</span></div><div class="s" style="color:#8b7cff">高级光照</div></div></div>
</div>
</div>

## 同父材质 · 不同子材质的对比

光看缩略图墙不够直观。下面挑三组**同一父材质规则下**的子材质做并排对比——它们共享完全相同的节点图，**唯一的差别就是几个暴露参数的取值**。这正是材质实例化的核心价值：一套母材质，靠参数派生出天差地别的视觉。

### 对比 A：大理石 vs 木纹（程序化纹理类）

同一套 `M_Procedural_Master`，靠噪声/条纹参数的此消彼长，一个变成各向同性的云斑大理石，一个变成有方向性的条带木纹。

<div class="ume">
<div class="vs">
<div class="col"><img src="img/r_proc_marble.png" alt="程序化大理石">
<div class="ch"><div class="nm">程序化大理石 <span class="bd bt">典型</span></div><div class="sm">白底深色纹理，低粗糙光泽</div></div>
<div class="pl">
<div class="r"><span>Noise Scale</span><b>3.0</b></div>
<div class="r"><span>Cell Blend</span><b>0.0 （纯 FBM）</b></div>
<div class="r"><span>Stripe Amount</span><b>0.0 （无条纹）</b></div>
<div class="r"><span>Pattern Contrast</span><b>2.5</b></div>
<div class="r"><span>Color A / B</span><b><span class="sw" style="background:#d9d7d1"></span>浅 / <span class="sw" style="background:#333840"></span>深</b></div>
<div class="r"><span>Roughness Base</span><b>0.15</b></div>
</div></div>
<div class="col"><img src="img/r_proc_wood.png" alt="程序化木纹">
<div class="ch"><div class="nm">程序化木纹 <span class="bd bb">边界</span></div><div class="sm">暖色木纹条带，沿 U 方向延伸</div></div>
<div class="pl">
<div class="r"><span>Noise Scale</span><b>6.0</b></div>
<div class="r"><span>Cell Blend</span><b>0.1</b></div>
<div class="r"><span>Stripe Amount</span><b>0.7 （强条纹）</b></div>
<div class="r"><span>Stripe Frequency</span><b>35.0</b></div>
<div class="r"><span>Color A / B</span><b><span class="sw" style="background:#40210f"></span>深棕 / <span class="sw" style="background:#996133"></span>暖棕</b></div>
<div class="r"><span>Roughness Base</span><b>0.4</b></div>
</div></div>
</div>
<div class="delta"><b>关键变量：</b>大理石 <code>Stripe Amount=0</code> → 纹理无方向性，由 FBM 噪声主导；木纹把 <code>Stripe Amount</code> 拉到 <b>0.7</b>、<code>Stripe Frequency=35</code>，叠加暖色双色渐变，立刻获得沿 U 轴的木质条带感。粗糙度也从 0.15（抛光石材）升到 0.4（哑光木面）。</div>
</div>

### 对比 B：多档赛璐珞 vs 线稿描边（风格化渲染类）

同一套 `M_Stylized_Master`，核心差异在 `Shade Bands`（色阶档数）与 `Outline` 一组参数——从"有立体感的多档卡通"一路推到"几乎无明暗、只剩粗黑描边的漫画线稿"。

<div class="ume">
<div class="vs">
<div class="col"><img src="img/r_sty_cel.png" alt="多档赛璐珞">
<div class="ch"><div class="nm">多档赛璐珞 <span class="bd bt">典型</span></div><div class="sm">冷色调多档过渡，接近半写实</div></div>
<div class="pl">
<div class="r"><span>Shade Bands</span><b>6 档</b></div>
<div class="r"><span>Lit / Shadow</span><b><span class="sw" style="background:#ccd9f2"></span>亮 / <span class="sw" style="background:#4d66a0"></span>暗</b></div>
<div class="r"><span>Rim Power</span><b>6.0</b></div>
<div class="r"><span>Rim Color</span><b><span class="sw" style="background:#80ccff"></span>冷蓝边缘光</b></div>
<div class="r"><span>Outline</span><b>仅阈值 0.8（细）</b></div>
</div></div>
<div class="col"><img src="img/r_sty_ink.png" alt="线稿描边">
<div class="ch"><div class="nm">线稿描边 <span class="bd be">极端</span></div><div class="sm">近乎纯白平涂 + 粗黑描边</div></div>
<div class="pl">
<div class="r"><span>Shade Bands</span><b>1 档（几乎无明暗）</b></div>
<div class="r"><span>Lit / Shadow</span><b><span class="sw" style="background:#faf9f5"></span>白 / <span class="sw" style="background:#d9d8d1"></span>近白</b></div>
<div class="r"><span>Outline Width</span><b>0.4 （粗）</b></div>
<div class="r"><span>Outline Threshold</span><b>0.5</b></div>
<div class="r"><span>Outline Color</span><b><span class="sw" style="background:#000"></span>纯黑</b></div>
</div></div>
</div>
<div class="delta"><b>层级差异：</b><code>Shade Bands</code> 从 <b>6 → 1</b> 是关键——6 档时 N·L 被量化成 6 级灰阶，仍有体积感；降到 1 档时明暗几乎消失，主体成平涂。同时把 <code>Outline Width</code> 从近乎无（仅靠阈值 0.8 的细线）放到 <b>0.4</b> 的粗黑边，整体从"赛璐珞角色"切换为"漫画线稿"。</div>
</div>

### 对比 C：丝绒布料 vs 肥皂泡虹彩（高级光照类）

同一套 `M_AdvLighting_Master`，分别只激活 **布料（Cloth/Fuzz）** 与 **虹彩（Iridescence）** 两组高级 BRDF 参数——一个是漫反射柔软的绒面、边缘绒毛泛光；一个是镜面光滑、随视角变换的彩虹薄膜。

<div class="ume">
<div class="vs">
<div class="col"><img src="img/r_adv_velvet.png" alt="丝绒布料">
<div class="ch"><div class="nm">丝绒布料 <span class="bd be">极端</span></div><div class="sm">深红丝绒，边缘绒毛泛光</div></div>
<div class="pl">
<div class="r"><span>Base Color</span><b><span class="sw" style="background:#66050f"></span>深红</b></div>
<div class="r"><span>Roughness</span><b>0.8 （粗糙漫反射）</b></div>
<div class="r"><span>Cloth Amount</span><b>1.0</b></div>
<div class="r"><span>Fuzz Color</span><b><span class="sw" style="background:#e6808f"></span>粉绒边</b></div>
<div class="r"><span>Iridescence</span><b>关闭</b></div>
</div></div>
<div class="col"><img src="img/r_adv_bubble.png" alt="肥皂泡虹彩">
<div class="ch"><div class="nm">肥皂泡虹彩 <span class="bd be">极端</span></div><div class="sm">暗底强虹彩，随视角变换彩带</div></div>
<div class="pl">
<div class="r"><span>Base Color</span><b><span class="sw" style="background:#0d0d14"></span>近黑</b></div>
<div class="r"><span>Roughness</span><b>0.05 （镜面光滑）</b></div>
<div class="r"><span>Iridescence</span><b>开启 · Bands 6</b></div>
<div class="r"><span>Iri. Intensity</span><b>2.5</b></div>
<div class="r"><span>Clear Coat</span><b>1.0</b></div>
</div></div>
</div>
<div class="delta"><b>光学表现对比：</b>丝绒走的是 <b>Cloth 模型</b>——高 <code>Roughness 0.8</code> 让主体漫反射柔和，靠 <code>Fuzz Color</code> 在掠射角生成绒毛泛光，质感"吸光"。肥皂泡反其道——<code>Roughness 0.05</code> 近乎镜面，<code>Enable Iridescence</code> 打开后由 N·V 相位差生成 6 条彩虹带（<code>Bands=6</code>、<code>Intensity=2.5</code>），再叠 <code>Clear Coat=1.0</code> 清漆，质感"反光 + 干涉"。同一母材质，一个吸光绒面、一个干涉镜面，光学行为完全相反。</div>
</div>

## 四维量化评测体系

评分引擎对每个父材质自动解析打分，四个维度各占 25%：

| 维度 | 权重 | 子项构成 |
|---|:--:|---|
| **节点复杂度** | 25% | 节点数（对数缩放）+ 最大层级深度 + 节点类型多样性 + 连边密度 |
| **参数化设计合理性** | 25% | 参数规模 + 分组组织 + 范围/UI 范围完整性 + 语义说明 + 默认值合法性 |
| **效果多样性** | 25% | 子实例数量 + 参数覆盖广度 + 采样类型熵（三类均衡=满）+ 取值离散度 |
| **技术正确性** | 25% | 连接完整性 + 数据类型匹配 + 参数绑定 + 输出引脚与 ShadingModel 一致性 − 孤立节点 |

其中**技术正确性**是 TA 视角的"硬规则"：每条连边端点必须存在、标注数据类型、参数必须真的绑定到 `*Parameter` 节点、输出引脚必须被当前 ShadingModel 支持（Unlit 不能接 Roughness、只有 ClearCoat 才有 ClearCoat 引脚）。

### 四维雷达：五类材质 vs 基线

<div class="ume">
<div style="text-align:center">
<svg viewBox="0 0 520 430" width="100%" style="max-width:520px"><polygon points="260.0,172.5 297.5,210.0 260.0,247.5 222.5,210.0" fill="none" stroke="#2a2f3a" stroke-width="1"/><polygon points="260.0,135.0 335.0,210.0 260.0,285.0 185.0,210.0" fill="none" stroke="#2a2f3a" stroke-width="1"/><polygon points="260.0,97.5 372.5,210.0 260.0,322.5 147.5,210.0" fill="none" stroke="#2a2f3a" stroke-width="1"/><polygon points="260.0,60.0 410.0,210.0 260.0,360.0 110.0,210.0" fill="none" stroke="#2a2f3a" stroke-width="1"/><line x1="260" y1="210" x2="260.0" y2="60.0" stroke="#2a2f3a" stroke-width="1"/><text x="260.0" y="33.0" fill="#9aa3b2" font-size="13" text-anchor="middle">节点复杂度</text><line x1="260" y1="210" x2="410.0" y2="210.0" stroke="#2a2f3a" stroke-width="1"/><text x="437.0" y="210.0" fill="#9aa3b2" font-size="13" text-anchor="middle">参数化设计</text><line x1="260" y1="210" x2="260.0" y2="360.0" stroke="#2a2f3a" stroke-width="1"/><text x="260.0" y="387.0" fill="#9aa3b2" font-size="13" text-anchor="middle">效果多样性</text><line x1="260" y1="210" x2="110.0" y2="210.0" stroke="#2a2f3a" stroke-width="1"/><text x="83.0" y="210.0" fill="#9aa3b2" font-size="13" text-anchor="middle">技术正确性</text><polygon points="260.0,90.3 410.0,210.0 260.0,332.4 119.0,210.0" fill="#8b7cff22" stroke="#8b7cff" stroke-width="2"/><polygon points="260.0,67.9 399.8,210.0 260.0,354.4 110.0,210.0" fill="#ffb84d22" stroke="#ffb84d" stroke-width="2"/><polygon points="260.0,97.2 396.6,210.0 260.0,349.5 110.0,210.0" fill="#5b9dff22" stroke="#5b9dff" stroke-width="2"/><polygon points="260.0,70.3 403.6,210.0 260.0,342.8 110.0,210.0" fill="#3ddc8422" stroke="#3ddc84" stroke-width="2"/><polygon points="260.0,87.6 396.6,210.0 260.0,327.8 116.0,210.0" fill="#ff7eb622" stroke="#ff7eb6" stroke-width="2"/><polygon points="260.0,192.0 260.0,210.0 260.0,222.0 170.0,210.0" fill="#6b728011" stroke="#6b7280" stroke-width="2" stroke-dasharray="5,4"/></svg>
<div class="legend">
<span><span class="dot" style="background:#5b9dff"></span>PBR 基础</span>
<span><span class="dot" style="background:#3ddc84"></span>程序化</span>
<span><span class="dot" style="background:#ff7eb6"></span>风格化</span>
<span><span class="dot" style="background:#ffb84d"></span>动态效果</span>
<span><span class="dot" style="background:#8b7cff"></span>高级光照</span>
<span><span class="dot" style="background:#6b7280"></span>基线(虚线)</span>
</div>
</div>
</div>

### 评分明细：套件均分 91.2，碾压基线 20.0

<div class="ume">
<h3>分维度评分（按总分排序，含基线对照）</h3>
<table>
<tr><th>材质</th><th>节点复杂度</th><th>参数化设计</th><th>效果多样性</th><th>技术正确性</th><th>总分</th><th>等级</th></tr>
<tr><td><b>动态效果类</b></td><td><div class="bar"><i style="width:94.7%;background:#3ddc84"></i></div>94.7</td><td><div class="bar"><i style="width:93.2%;background:#3ddc84"></i></div>93.2</td><td><div class="bar"><i style="width:96.3%;background:#3ddc84"></i></div>96.3</td><td><div class="bar"><i style="width:100%;background:#3ddc84"></i></div>100</td><td class="n"><b style="color:#3ddc84">96.1</b></td><td><span class="g gS">S</span></td></tr>
<tr><td><b>程序化纹理类</b></td><td><div class="bar"><i style="width:93.1%;background:#3ddc84"></i></div>93.1</td><td><div class="bar"><i style="width:95.7%;background:#3ddc84"></i></div>95.7</td><td><div class="bar"><i style="width:88.5%;background:#3ddc84"></i></div>88.5</td><td><div class="bar"><i style="width:100%;background:#3ddc84"></i></div>100</td><td class="n"><b style="color:#3ddc84">94.3</b></td><td><span class="g gS">S</span></td></tr>
<tr><td><b>PBR 基础类</b></td><td><div class="bar"><i style="width:75.2%;background:#5b9dff"></i></div>75.2</td><td><div class="bar"><i style="width:91.1%;background:#3ddc84"></i></div>91.1</td><td><div class="bar"><i style="width:93%;background:#3ddc84"></i></div>93.0</td><td><div class="bar"><i style="width:100%;background:#3ddc84"></i></div>100</td><td class="n"><b style="color:#3ddc84">89.8</b></td><td><span class="g gA">A</span></td></tr>
<tr><td><b>高级光照类</b></td><td><div class="bar"><i style="width:79.8%;background:#5b9dff"></i></div>79.8</td><td><div class="bar"><i style="width:100%;background:#3ddc84"></i></div>100</td><td><div class="bar"><i style="width:81.6%;background:#5b9dff"></i></div>81.6</td><td><div class="bar"><i style="width:94%;background:#3ddc84"></i></div>94.0</td><td class="n"><b style="color:#3ddc84">88.8</b></td><td><span class="g gA">A</span></td></tr>
<tr><td><b>风格化渲染类</b></td><td><div class="bar"><i style="width:81.6%;background:#5b9dff"></i></div>81.6</td><td><div class="bar"><i style="width:91.1%;background:#3ddc84"></i></div>91.1</td><td><div class="bar"><i style="width:78.5%;background:#5b9dff"></i></div>78.5</td><td><div class="bar"><i style="width:96%;background:#3ddc84"></i></div>96.0</td><td class="n"><b style="color:#3ddc84">86.8</b></td><td><span class="g gA">A</span></td></tr>
<tr style="opacity:.6"><td>基线·贴图直连</td><td><div class="bar"><i style="width:12%;background:#6b7280"></i></div>12.0</td><td><div class="bar"><i style="width:0%;background:#6b7280"></i></div>0.0</td><td><div class="bar"><i style="width:8%;background:#6b7280"></i></div>8.0</td><td><div class="bar"><i style="width:60%;background:#6b7280"></i></div>60.0</td><td class="n"><b>20.0</b></td><td><span class="g gD">D</span></td></tr>
</table>
</div>

## 结论与改进方向

**结论：** 五类父材质总分 86.8–96.1，相对基线 20.0 实现约 **4.3×–4.8× 跃迁**，套件均分 91.2。这说明 agent 确实具备从"资产搬运"到"材质体系设计"的能力，覆盖了参数化（T1）、程序化（T2）、自定义光照与动态（T3）、高级 BRDF（T4）的完整阶梯。技术正确性整体优异（4/5 满分），节点连接逻辑与数据类型匹配可靠。

几个值得记的观察：

- **动态效果类拿下最高分（96.1）** —— 节点最密、采样最均衡（typical/boundary/extreme 三类齐全）、技术零缺陷。
- **高级光照类参数化满分（100）** —— 16 个参数、全分组、全语义；但因把 SSS / Cloth / 各向异性 / 虹彩并联进单一 ClearCoat 母材质，部分高级引脚是"声明型"的，导致节点复杂度评分略低。
- **风格化类效果多样性偏低（78.5）** —— 4 个子实例都收敛在卡通族里，视觉差异度不够。

**下一轮的改进方向：**

1. 风格化类增加半色调 / 水墨 / 像素化等差异更大的子风格，拉高效果多样性。
2. 把"统一母材质 + StaticSwitch"拆成各 ShadingModel 独立母材质，让 SSS / Cloth / 各向异性的输出引脚与模型严格一致。
3. **接入真实渲染**：当前预览是程序化近似合成，可对接源码版 UE，用 Python 批量实例化 MIC 并 HighResShot 出真实渲染图替换占位。
4. 引入指令数 / 采样器数 / 是否含 CustomExpression 的**性能预算维度**，平衡"复杂度"与运行时开销。

> 一句话总结：从"能不能把贴图连上"，到"能不能设计一整套带阶梯的材质体系"——这条评测线把 agent 的材质能力从 20 分推到了 91 分。
