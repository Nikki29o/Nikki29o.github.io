---
title: "UE 材质评测·真机渲染版：23 个材质实例的 Unreal Engine 实拍"
date: 2026-06-17T19:30:00+08:00
draft: false
description: "上一篇评测的渲染图是 Python 程序化近似合成的；这一篇推倒重来，在源码编译的 Unreal Engine 5.7.3 里真实重建 5 个母材质、派生 23 个 MaterialInstance，逐个上预览球、统一布光、HighResShot 出图。本文是同一套评测体系的真机渲染版本。"
tags: ["Unreal Engine", "Material", "Shader", "Technical Art", "AI Evaluation", "PBR", "Rendering"]
categories: ["Tool Review"]
---

[上一篇《UE 材质蓝图复杂度评测》](../ue-material-complexity-eval/) 里我坦白过一件事：那 23 张材质预览图**不是 UE 渲染的**，而是用 Python + numpy 按参数语义程序化合成的近似图。这一篇把这个坑填上——在**源码编译的 Unreal Engine 5.7.3** 里真实重建全部材质，逐个渲染、出图。评测体系、分类、参数完全沿用，只把"示意图"换成"真机实拍"。

<!--more-->

> 这是同一评测的**独立真机渲染版本**，旧版（程序化近似图）依然保留可访问，方便对照"近似 vs 真实"的差距。

<style>
.umr{--bg:#0f1115;--panel:#171a21;--panel2:#1e222b;--border:#2a2f3a;--text:#e6e9ef;--muted:#9aa3b2;--good:#3ddc84;--warn:#ffb84d;background:var(--bg);color:var(--text);border:1px solid var(--border);border-radius:14px;padding:22px;margin:22px 0;font-family:"Segoe UI","PingFang SC","Microsoft YaHei",sans-serif}
.umr *{box-sizing:border-box}
.umr h3{color:#fff;border:0;margin:6px 0 14px;font-size:17px}
.umr .kpis{display:grid;grid-template-columns:repeat(4,1fr);gap:10px}
.umr .kpi{background:var(--panel2);border:1px solid var(--border);border-radius:10px;padding:13px}
.umr .kpi .v{font-size:23px;font-weight:700;color:#fff}
.umr .kpi .l{font-size:11px;color:var(--muted);margin-top:3px}
.umr .steps{counter-reset:s;list-style:none;padding:0;margin:0}
.umr .steps li{position:relative;padding:10px 0 10px 42px;border-top:1px solid var(--border);font-size:13.5px;color:var(--text)}
.umr .steps li:before{counter-increment:s;content:counter(s);position:absolute;left:0;top:9px;width:28px;height:28px;border-radius:8px;background:#5b9dff;color:#fff;text-align:center;line-height:28px;font-weight:700}
.umr .steps b{color:#fff}
.umr .gallery{display:grid;grid-template-columns:repeat(auto-fill,minmax(150px,1fr));gap:12px}
.umr .gi{background:var(--panel2);border:1px solid var(--border);border-radius:10px;overflow:hidden}
.umr .gi img{width:100%;display:block;aspect-ratio:1;object-fit:cover;background:#0a0b0e;margin:0}
.umr .gi .c{padding:7px 9px}
.umr .gi .t{font-size:11.5px;font-weight:600;color:#fff}
.umr .gi .s{font-size:9.5px;margin-top:2px}
.umr .bd{display:inline-block;font-size:9px;padding:1px 6px;border-radius:8px;margin-left:4px}
.umr .bt{background:#27405e;color:#a9cdf5}.umr .bb{background:#5e4d27;color:#f5dca9}.umr .be{background:#5e2740;color:#f5a9cd}
.umr .vs{display:grid;grid-template-columns:1fr 1fr;gap:14px}
.umr .vs .col{background:var(--panel2);border:1px solid var(--border);border-radius:11px;overflow:hidden}
.umr .vs .col>img{width:100%;display:block;aspect-ratio:1;object-fit:cover;background:#0a0b0e;margin:0}
.umr .vs .ch{padding:10px 12px 4px}.umr .vs .ch .nm{font-size:14px;font-weight:700;color:#fff}
.umr .vs .ch .sm{font-size:11px;color:var(--muted);margin-top:2px}
.umr .vs .pl{padding:2px 12px 12px;font-size:11.5px}
.umr .vs .pl .r{display:flex;justify-content:space-between;gap:8px;padding:4px 0;border-top:1px solid var(--border)}
.umr .vs .pl .r span{color:var(--muted)}.umr .vs .pl .r b{color:#fff;font-variant-numeric:tabular-nums}
.umr .sw{display:inline-block;width:11px;height:11px;border-radius:3px;vertical-align:-1px;margin-right:3px;border:1px solid #3a3f4a}
.umr .delta{background:#1a1d24;border-left:3px solid var(--warn);border-radius:0 8px 8px 0;padding:9px 12px;margin-top:12px;font-size:12.5px;color:var(--muted)}
.umr .delta b{color:var(--text)}
.umr .gw{overflow-x:auto;background:#14171d;border:1px solid var(--border);border-radius:11px;padding:10px}
.umr .gw img{max-width:none;margin:0}
@media(max-width:760px){.umr .kpis{grid-template-columns:repeat(2,1fr)}.umr .vs{grid-template-columns:1fr 1fr}}
</style>

## 这次真的是 UE 渲染

每张图都来自 `D:\UnrealEngine`（GitHub 源码编译的 UE 5.7.3）的编辑器视口截图——能看到**真实的 HDRI 天空、方向光投影、地面网格反射、天光环境光**。预览物用的是 UE 自带的材质球 `SM_MatPreviewMesh_01`。

<div class="umr">
<div class="kpis">
<div class="kpi"><div class="v">5</div><div class="l">UE 母材质 (真实编译)</div></div>
<div class="kpi"><div class="v">23</div><div class="l">MaterialInstance 实例</div></div>
<div class="kpi"><div class="v">640²</div><div class="l">HighResShot 分辨率</div></div>
<div class="kpi"><div class="v">5.7.3</div><div class="l">UE 版本 (源码编译)</div></div>
</div>
</div>

### 渲染管线：从 JSON 到 UE 实拍

<div class="umr">
<ol class="steps">
<li><b>重建母材质。</b>用 <code>MaterialEditingLibrary</code> 在 <code>/Game/Eval/Masters</code> 真实搭建 5 个母材质的节点图，暴露参数名与评测 JSON 严格一致。风格化类设 <b>Unlit</b>、动态类设 <b>Masked + 双面</b>、高级光照类设 <b>ClearCoat</b>。</li>
<li><b>派生实例。</b>对 23 个子实例创建 <code>MaterialInstanceConstant</code>，按 overrides 调 <code>set_scalar / vector / static_switch_parameter_value</code>。</li>
<li><b>搭场景。</b>预览球 + 地面 + 主光（暖白）+ 天光（实时捕获）+ 大气天空 + 冷色补光，定位视口相机。</li>
<li><b>逐个渲染。</b>遍历 23 个实例：套用材质 → 等着色器编译/天光稳定 → 控制台 <code>HighResShot 640x640</code>。</li>
<li><b>取回出图。</b>轮询截图目录、按顺序映射回 <code>render_ref</code>，归集成 23 张 UE 实拍。</li>
</ol>
</div>

> 工程上踩到的真坑：UE Python 的 `take_high_res_screenshot(filename)` 在连续调用时异步写盘会**丢文件**；最终改用控制台 `HighResShot`（它会按 `HighresScreenshotNNNNN.png` 稳定递增），再按固定顺序重命名映射，才拿全 23 张。

## 五类父材质节点蓝图

材质的图结构（按层级深度从左到右，★=暴露参数）和评测体系与上一篇一致，这里一并附上，方便对照每张实拍对应的母材质：

**① PBR 基础类** · DefaultLit · 25 节点 / 11 参数
<div class="umr"><div class="gw"><img src="graphs/M_PBR_Base_Master.svg" alt="PBR 节点蓝图" style="width:1962px"></div></div>

**② 程序化纹理类** · 33 节点 / 12 参数（无贴图，纯数学）
<div class="umr"><div class="gw"><img src="graphs/M_Procedural_Master.svg" alt="程序化 节点蓝图" style="width:2550px"></div></div>

**③ 风格化渲染类** · Unlit · 27 节点 / 11 参数
<div class="umr"><div class="gw"><img src="graphs/M_Stylized_Master.svg" alt="风格化 节点蓝图" style="width:2158px"></div></div>

**④ 动态效果类** · Masked · 36 节点 / 12 参数
<div class="umr"><div class="gw"><img src="graphs/M_DynamicFX_Master.svg" alt="动态 节点蓝图" style="width:2354px"></div></div>

**⑤ 高级光照类** · ClearCoat · 29 节点 / 16 参数
<div class="umr"><div class="gw"><img src="graphs/M_AdvLighting_Master.svg" alt="高级光照 节点蓝图" style="width:2158px"></div></div>

## 23 个实例 · UE 实拍总览

全部 23 张均为 UE 编辑器视口 HighResShot。注意每张都有真实的天空反射与投影——这是程序化近似图给不了的。

<div class="umr">
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

## 同父材质 · 不同子材质对比（UE 实拍）

三组并排对比，共享同一母材质，仅参数取值不同——这次差异是真机渲染出来的。

### 大理石 vs 木纹（程序化纹理类）

<div class="umr">
<div class="vs">
<div class="col"><img src="img/r_proc_marble.png">
<div class="ch"><div class="nm">程序化大理石 <span class="bd bt">典型</span></div><div class="sm">白底黑斑，低粗糙光泽</div></div>
<div class="pl">
<div class="r"><span>Noise Scale</span><b>3.0</b></div>
<div class="r"><span>Stripe Amount</span><b>0.0（无条纹）</b></div>
<div class="r"><span>Pattern Contrast</span><b>2.5</b></div>
<div class="r"><span>Color A / B</span><b><span class="sw" style="background:#d9d7d1"></span>浅 / <span class="sw" style="background:#333840"></span>深</b></div>
<div class="r"><span>Roughness Base</span><b>0.15</b></div>
</div></div>
<div class="col"><img src="img/r_proc_wood.png">
<div class="ch"><div class="nm">程序化木纹 <span class="bd bb">边界</span></div><div class="sm">暖色条带，沿 U 方向</div></div>
<div class="pl">
<div class="r"><span>Noise Scale</span><b>6.0</b></div>
<div class="r"><span>Stripe Amount</span><b>0.7（强条纹）</b></div>
<div class="r"><span>Stripe Frequency</span><b>35.0</b></div>
<div class="r"><span>Color A / B</span><b><span class="sw" style="background:#40210f"></span>深棕 / <span class="sw" style="background:#996133"></span>暖棕</b></div>
<div class="r"><span>Roughness Base</span><b>0.4</b></div>
</div></div>
</div>
<div class="delta"><b>关键变量：</b><code>Stripe Amount</code> 0→0.7 让纹理从各向同性斑块变成方向性条带，配合暖色双色渐变与更高的 Noise Scale，木质感成立；粗糙度 0.15→0.4 也让表面从抛光石材转为哑光木面。</div>
</div>

### 多档赛璐珞 vs 线稿描边（风格化渲染类）

<div class="umr">
<div class="vs">
<div class="col"><img src="img/r_sty_cel.png">
<div class="ch"><div class="nm">多档赛璐珞 <span class="bd bt">典型</span></div><div class="sm">冷色多档过渡</div></div>
<div class="pl">
<div class="r"><span>Shade Bands</span><b>6 档</b></div>
<div class="r"><span>Lit / Shadow</span><b><span class="sw" style="background:#ccd9f2"></span>亮 / <span class="sw" style="background:#4d66a0"></span>暗</b></div>
<div class="r"><span>Rim Power</span><b>6.0</b></div>
<div class="r"><span>Outline</span><b>细（阈值 0.8）</b></div>
</div></div>
<div class="col"><img src="img/r_sty_ink.png">
<div class="ch"><div class="nm">线稿描边 <span class="bd be">极端</span></div><div class="sm">近纯白平涂 + 粗黑边</div></div>
<div class="pl">
<div class="r"><span>Shade Bands</span><b>1 档（几乎无明暗）</b></div>
<div class="r"><span>Lit / Shadow</span><b><span class="sw" style="background:#faf9f5"></span>白 / <span class="sw" style="background:#d9d8d1"></span>近白</b></div>
<div class="r"><span>Outline Width</span><b>0.4（粗）</b></div>
<div class="r"><span>Outline Color</span><b><span class="sw" style="background:#000"></span>纯黑</b></div>
</div></div>
</div>
<div class="delta"><b>层级差异：</b><code>Shade Bands</code> 6→1 是核心——6 档保留体积感，1 档几乎平涂；再把 <code>Outline Width</code> 放粗，整体从"赛璐珞角色"切到"漫画线稿"。这两张都是 Unlit 自定义光照在 UE 里实拍。</div>
</div>

### 丝绒布料 vs 肥皂泡虹彩（高级光照类）

<div class="umr">
<div class="vs">
<div class="col"><img src="img/r_adv_velvet.png">
<div class="ch"><div class="nm">丝绒布料 <span class="bd be">极端</span></div><div class="sm">深红绒面，边缘泛光</div></div>
<div class="pl">
<div class="r"><span>Base Color</span><b><span class="sw" style="background:#66050f"></span>深红</b></div>
<div class="r"><span>Roughness</span><b>0.8（漫反射）</b></div>
<div class="r"><span>Cloth Amount</span><b>1.0</b></div>
<div class="r"><span>Fuzz Color</span><b><span class="sw" style="background:#e6808f"></span>粉绒边</b></div>
</div></div>
<div class="col"><img src="img/r_adv_bubble.png">
<div class="ch"><div class="nm">肥皂泡虹彩 <span class="bd be">极端</span></div><div class="sm">暗底镜面 + 虹彩薄膜</div></div>
<div class="pl">
<div class="r"><span>Base Color</span><b><span class="sw" style="background:#0d0d14"></span>近黑</b></div>
<div class="r"><span>Roughness</span><b>0.05（镜面）</b></div>
<div class="r"><span>Iridescence</span><b>开启 · Bands 6</b></div>
<div class="r"><span>Clear Coat</span><b>1.0</b></div>
</div></div>
</div>
<div class="delta"><b>光学对比：</b>丝绒高 <code>Roughness 0.8</code> + Fuzz 绒毛在掠射角泛光，质感"吸光"；肥皂泡 <code>Roughness 0.05</code> 近镜面 + 虹彩薄膜（Fresnel 相位差生成彩带）+ 清漆层，质感"反光 + 干涉"。同一 ClearCoat 母材质，两种相反的光学行为。</div>
</div>

## 近似 vs 真实：这次补上了什么

| 维度 | 上一篇（程序化近似） | 本篇（UE 实拍） |
|---|---|---|
| 着色 | numpy 手写 Blinn-Phong | UE 物理 BRDF / 各 ShadingModel |
| 环境光照 | 单点方向光 | HDRI 天空 + 天光 + 大气 |
| 反射 / 投影 | 无 | 真实天空反射、地面投影 |
| 材质实例 | 仅参数→颜色映射 | 真实 MaterialInstanceConstant |
| 引擎一致性 | 与 UE 无关 | 即评测里声明的 UE 行为 |

**结论：** 评测体系（5 类 / 23 实例 / 四维评分）保持不变，但渲染证据从"示意"升级为"真机"。需要复现的话，全部脚本（母材质重建 `build_eval_materials.py`、批量截图 `init_unreal.py`）都在评测工程里。

> 想看分类设计、子实例采样规则与四维量化评分的完整论述，请回看 [上一篇评测正文](../ue-material-complexity-eval/)——本篇专注于"把图换成真的"。
