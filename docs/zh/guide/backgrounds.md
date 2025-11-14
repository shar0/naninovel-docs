# 背景

与 [角色](/zh/guide/characters) 不同，背景是用于表示场景“后层”的演出元素：例如地点、风景、景观或任何应始终出现在角色“后方”的元素。

一个背景演出元素由名称、外观、可见性以及变换属性（位置、旋转、缩放）定义。背景可以随时间改变其外观、可见性与变换。

背景的行为可通过 `Naninovel -> Configuration -> Backgrounds` 上下文菜单进行配置；有关可用选项，请参阅 [配置指南](/zh/guide/configuration#backgrounds)。背景资源管理器可通过 `Naninovel -> Resources -> Backgrounds` 上下文菜单访问。

![](https://i.gyazo.com/cccd08280dac72d199ea3465bc167a22.gif)

在 Naninovel 脚本中，背景主要通过 [@back] 指令进行控制：

```nani
; 将ID为 MainBackground 的主背景外观设置为 `River`。
@back River

; 与上相同，但同时使用 `RadialBlur` 过渡效果。
@back River.RadialBlur
```

背景的处理方式与角色略有不同，以更好地适应传统视觉小说的游戏流程。在大多数情况下，场景中通常只有一个背景演出元素，并不断切换不同外观。为避免在脚本中重复输入相同的演出元素 ID，可以只提供背景外观和可选的过渡类型参数，系统会默认作用于 `MainBackground` 演出元素。若需要控制其他背景演出元素，可通过 `id` 参数显式指定：

```nani
; 假设存在一个 `CityVideo` 演出元素，包含 `Night` 和 `Day` 两个视频片段。

; 播放白天片段的视频背景。
@back Day id:CityVideo

; 使用波纹（ripple）效果切换到夜晚片段。 
@back Night.Ripple id:CityVideo

; 隐藏视频背景。
@hide CityVideo
```

主背景（MainBackground）会在背景资源管理器中默认创建，且无法重命名或删除；但你可以自由修改其参数（实现方式、中心点、PPU 等）。

以下视频教程概述了背景演出元素的用法：

![](https://www.youtube.com/watch?v=X2iyGSCpnJs)

上哔哩哔哩观看 👉 [转载版视频](https://www.bilibili.com/video/BV1U9CuBJESi/)。

## 演出元素记录

如果你拥有大量背景或外观，通过编辑器菜单逐一分配会比较麻烦，可以使用演出元素记录资源（`Create -> Naninovel -> Actor Record -> Background`）。演出元素记录支持多选编辑，并可通过文件夹组织。下方视频展示了一个示例：

![](https://www.youtube.com/watch?v=2YP-36THHvk)

上哔哩哔哩观看 👉 [转载版视频](https://www.bilibili.com/video/BV1SDCuBHEHW/)。

要将外观资源与演出元素记录关联，请使用 [可寻址资源系统](/zh/guide/resource-providers#addressable)。例如，要将 “Beach” 外观与 “MainBackground” 背景关联，请将纹理资源地址设为 `Naninovel/Backgrounds/MainBackground/Beach`，并添加 `Naninovel` 标签。有关使用可寻址资源提供程序的详细信息，请参阅 [资源提供程序文档](/zh/guide/resource-providers#addressable)。

## Z 轴顺序

当同时显示多个背景时，它们往往会互相遮挡：

```nani
@back id:1
@back id:2
```

— 当背景 `1` 和 `2` 都是全屏不透明纹理时，后添加的会完全覆盖前一个。若想再次显示第一个，可以隐藏另一个或改变其 z 轴位置（深度）以调整绘制顺序：

```nani
; 隐藏背景 2 来重新显示第一个背景
@back id:2 !visible
; 也可以使用专用指令来隐藏演出元素
@hide 2

; 或者，改变 z 轴位置
@back id:1 pos:,,98
@back id:2 pos:,,99
```

较高的 z 轴位置表示距离摄像机更远，因此第一个演出元素若被放置在更靠近摄像机的位置，将会渲染在其他对象之上。

默认情况下，背景会以特定的 z 偏移量放置，以使其显示在其他演出元素类型之后。可以通过背景设置中的 `Z Offset` 属性来更改该偏移值。

为了防止 z-fighting 问题，背景在首次添加（显示）时会沿 z 轴进一步分层偏移。该偏移由 `Z Step` 设置控制。

## 匹配模式

当 [摄像机](https://docs.unity3d.com/Manual/class-Camera.html) 以正交模式渲染且背景演出元素配置中的 `Match Mode` 未被禁用时，演出元素将尝试根据当前屏幕尺寸自动调整自身大小。这用于处理显示 [宽高比](https://en.wikipedia.org/wiki/Aspect_ratio_(image)) 与背景图像不同的情况。如果关闭匹配功能且宽高比不一致，将出现“黑边”。

![](https://i.gyazo.com/46619a08e3b91441cf30800185932963.png)

对于独立版（PC、Mac、Linux）构建，可以在 [Player Settings](https://docs.unity3d.com/Manual/class-PlayerSettingsStandalone.html#Resolution) 中限制可用的宽高比；但在网页、主机和移动平台上则无法做到，应用程序必须根据目标设备进行适配。

以下匹配模式可为每个背景演出元素设置（通用实现除外）：

| 模式    | 描述 |
|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Crop    | 背景将始终占据整个相机视锥，无论显示器纵横比如何，都不会出现黑边；但背景的部分区域可能被裁剪。此模式为新建背景演出元素的默认设置。 |
| Fit     | 背景区域将始终完全可见，但当纵横比不同时时会出现黑边。 |
| Custom  | 允许使用自定义比例匹配宽度或高度。该比例由 `Custom Match Ratio` 控制：最小值（0）匹配宽度忽略高度，最大值（1）则相反。 |
| Disable | 不进行任何匹配。 |

::: tip
如果你想为通用或自定义背景实现类似的匹配功能，可参考论坛上的[按屏幕缩放示例](https://discord.com/channels/545676116871086080/1369983634236379240)。
:::

## 姿势

每个背景都有 `Poses` 属性，用于指定具名状态（姿势）。

姿势名称可作为外观在 [@back] 指令中使用，一次性应用姿势中选定的所有参数，而无需在指令参数中逐个指定。

```nani
; 当主背景定义了 `Day` 姿势时，  
; 会应用该姿势中指定的所有参数。
@back Day

; 与上述相同，但作用于 ID 为 `City` 的背景演出元素，  
; 并使用 `DropFade` 过渡，持续 3 秒。
@back Day id:City transition:DropFade time:3
```

注意，当姿势作为外观使用时，你仍然可以覆写单独的参数，例如：

```nani
; 当主背景定义了 `Day` 姿势时，  
; 会应用姿势中指定的所有参数，  
; 但 `tint` 会被指令中的参数覆盖。
@back Day tint:#ff45cb
```

## 占位背景

占位实现是默认背景实现，用于在你还没有任何用于表示背景的视觉资源时进行剧情草稿编写。它会在运行时程序化生成背景外观，让你在编写剧情时能清楚知道当前显示的是哪个背景。下面是一个占位背景 “EveningScene” 的示例，并叠加了几个[占位角色](/zh/guide/characters#placeholder-characters)。

![](https://i.gyazo.com/cebb0506d3743e2e1b20b1d3c214239a.png)

虽然 Naninovel 会自动生成背景占位图，你仍然可以在背景编辑器的 `Placeholder Appearances` 列表中为特定外观自定义样式。

![](https://i.gyazo.com/183dcc86fbf0d01de49d85d45686571f.png)

## 精灵背景

背景演出元素的 Sprite 实现是最常见且最简单的方式；它使用一组包裹在四边形网格（sprite）上的[纹理](https://docs.unity3d.com/Manual/Textures.html)资源来表示背景外观。纹理可以基于 `.jpg`、`.png`、`.tiff`、`.psd` 或 Unity [支持的任意图像格式](https://docs.unity3d.com/Manual/ImportingTextures)。

::: tip
选择最适合你工作流程的文件格式即可。在构建游戏时，Unity 会自动将资源（纹理、音频、视频等）转换为目标平台最合适的格式，因此项目中使用何种格式存储原始资源并不会影响最终效果。详见 Unity 官方的[资产工作流文档](https://docs.unity3d.com/Manual/AssetWorkflow)。
:::

Sprite 背景在场景中的初始（未缩放）大小取决于参考分辨率（摄像机配置）、背景的 `Pixels Per Unit` 属性（为每个背景分别设置）以及源纹理的分辨率。

Naninovel 默认会尝试让背景覆盖整个摄像机视锥，因此请确保源纹理的宽高比与参考分辨率一致；更多信息可参考[匹配模式指南](/zh/guide/backgrounds#match-mode)。

::: tip
在开始制作美术资源（角色和背景）前，与团队明确参考分辨率。这样美术成员就能按照正确的尺寸制作资源，你也无需在后期修正它们。
:::

## 切片精灵背景

基于开源的 [SpriteDicing](https://github.com/elringus/SpriteDicing) 包构建，`DicedSpriteBackground` 实现允许在背景纹理包含大量相似区域时，通过复用纹理块，大幅减少构建体积与显存占用。

切片背景的工作方式与切片角色实现几乎完全相同；设置与使用方法请参见[切片角色指南](/zh/guide/characters.html#diced-sprite-characters)。

## 视频背景

视频背景使用循环播放的 [VideoClip](https://docs.unity3d.com/Manual/class-VideoClip) 资源作为该背景外观。

各平台支持的视频格式请参考 Unity 的[视频源兼容性文档](https://docs.unity3d.com/Manual/VideoSources-FileCompatibility.html)。若使用带 α 通道（透明度）的视频，请参考 Unity 关于[透明视频支持格式的说明](https://docs.unity3d.com/Manual/VideoTransparency.html)。

::: info NOTE
当在视频资源导入设置中关闭 `Transcode` 时，该视频可能在某些平台无法播放。如果构建后的游戏中视频无法播放，请尝试开启转码选项后重新构建。

![](https://i.gyazo.com/9c3fb59dc8ebb2fbd0f5a5e79542e11f.png)
:::

::: tip EXAMPLE
如果你在实现无缝循环时遇到问题，请确保视频的首帧和末帧完全一致，并使用兼容的编码设置；可参考我们的[视频演出元素示例](/zh/guide/samples#video-actor)。
:::

若希望某个外观不进行循环播放，可在外观名称后追加 `NoLoop`（大小写不敏感）。

### WebGL 限制

在 WebGL 平台上，Unity 的视频播放器只能以 **Streaming 模式**工作，因此所有视频资源在构建 WebGL 版本时都会被复制到 `Assets/StreamingAssets/Backgrounds` 文件夹中。**StreamingAssets** 文件夹也会出现在构建输出目录中；发布部署时必须保留该文件夹，并确保你的 Web 服务器允许读取其中的数据。

被复制的视频文件不会被 Unity 转码（即使开启了转码选项也不会生效），因此源视频文件必须**本身就是浏览器可播放的格式**；或者你也可以在构建后手动替换游戏目录中的视频。下面是我们 WebGL Demo 使用的视频背景文件的详细元数据示例：

~~~
Container : MPEG-4
Container profile : Base Media
Container codec ID : isom (isom/iso2/avc1/mp41)
Format : AVC
Format/Info : Advanced Video Codec
Format profile : High@L4
Format settings, CABAC : Yes
Format settings, RefFrames : 4 frames
Codec ID : avc1
Codec ID/Info : Advanced Video Coding
Bit rate : 3 196 kb/s
Width : 1 920 pixels
Height : 1 080 pixels
Display aspect ratio : 16:9
Frame rate mode : Constant
Frame rate : 25.000 FPS
Color space : YUV
Chroma subsampling : 4:2:0
Bit depth : 8 bits
Scan type : Progressive
Writing library : x264 core 148 r2795 aaa9aa8
Encoding settings : cabac=1 / ref=3 / deblock=1:0:0 / analyse=0x3:0x113 / me=hex / subme=7 / psy=1 / psy_rd=1.00:0.00 / mixed_ref=1 / me_range=16 / chroma_me=1 / trellis=1 / 8x8dct=1 / cqm=0 / deadzone=21,11 / fast_pskip=1 / chroma_qp_offset=-2 / threads=12 / lookahead_threads=2 / sliced_threads=0 / nr=0 / decimate=1 / interlaced=0 / bluray_compat=0 / constrained_intra=0 / bframes=3 / b_pyramid=2 / b_adapt=1 / b_bias=0 / direct=1 / weightb=1 / open_gop=0 / weightp=2 / keyint=250 / keyint_min=25 / scenecut=40 / intra_refresh=0 / rc_lookahead=40 / rc=crf / mbtree=1 / crf=23.0 / qcomp=0.60 / qpmin=0 / qpmax=69 / qpstep=4 / ip_ratio=1.40 / aq=1:1.00
~~~

如果你使用的格式不是 mp4（例如 webm），请在资源提供器配置中通过 `Video Stream Extension` 属性设置所托管文件的扩展名。

![](https://i.gyazo.com/b3eb1ab2af513e6a131347d6e5e455e5.png)

## 分层背景

分层背景实现允许你将多个精灵（图层）组合成一个背景，并在运行时通过 naninovel 脚本单独开关每个图层。

::: tip
分层演出元素（Layered Actor）实现目前是最灵活的：支持所有渲染特性（相比 Generic 实现更加全面）。即使你不想使用图层表达式，而是希望通过 Unity Animator 或自定义系统控制外观；或需要渲染粒子系统、第三方渲染器等复杂对象，也请优先考虑分层演出元素中的 **Render Only** 和 **Camera Rendering** 模式，只有在不满足需求时再选择 Generic 或自定义实现。
:::

要创建一个分层背景 prefab，使用 `Create -> Naninovel -> Background -> Layered` 菜单。进入 [Prefab 编辑模式](https://docs.unity3d.com/Manual/EditingInPrefabMode.html) 后即可组合图层。默认会创建若干层和组，你可以自由使用或删除、添加其他结构。

分层背景与[分层角色](/zh/guide/characters#layered-characters)的逻辑几乎相同；请参考角色文档了解设置方式以及如何在 naninovel 脚本中控制它们。

请注意：[@back] 指令中的无名参数表示的是**外观（appearance）+ 过渡类型（transition）**，而不是像 [@char] 那样的“ID + appearance”。因此在使用图层组合表达式时，应按如下方式编写：

```nani
; 给定名为 "LayeredForest" 的背景演出元素
@back Group>Layer,Other/Group+Layer,-RootLayer.TransitionType id:LayeredForest
```

## 通用背景

通用背景是最灵活的背景演出元素实现方式。它基于一个预制体，并在根对象上挂载了 `Generic Background Behaviour` 组件。外观切换以及其它所有背景参数都会被转发为 [Unity Events](https://docs.unity3d.com/Manual/UnityEvents.html)，允许你以任意方式实现底层对象的具体行为。

![](https://i.gyazo.com/6483ef3e84549c1bbfbdffc6556308ea.png)

::: info NOTE
通用演出元素实现只负责把场景脚本中的事件往外转发，至于底层对象应该如何响应外观变化、可见性变化、如何适配屏幕比例变化等，全都由你自行实现。不要期待大部分内置背景特性会自动适配通用实现。
:::

要基于模板创建通用背景预制体，使用 `Create -> Naninovel -> Background -> Generic` 上下文菜单。

通用背景与通用角色非常相似；你可以参考教程视频（将一个带动画的 3D 模型设置为通用角色）来了解可能的使用方式。注意，该视频使用的是旧版本 Naninovel，其中部分属性和组件名称已变更；请参照上文文档获取最新信息。

![](https://www.youtube.com/watch?v=HPxhR0I1u2Q)

上哔哩哔哩观看 👉 [转载版视频](https://www.bilibili.com/video/BV15pCgB8EpF/)。

::: tip
Unity 的 `Animator` 组件在物体启用/禁用发生在同一帧时，可能无法成功响应 `SetTrigger`。如果你像视频示例中那样使用 `GameObject.SetActive` 控制可见性，建议改为仅启用/禁用具有渲染器的子对象。
:::

::: tip EXAMPLE
参考 [通用演出元素示例](/zh/guide/samples#generic-actor)，其中演示了如何使用通用背景实现承载动画精灵的背景。
:::

## 场景背景

你可以使用一个 [Unity 场景](https://docs.unity3d.com/Manual/CreatingScenes) 作为背景，通过 Scene Backgrounds 实现方式来呈现。

场景背景的配置中有一个 `Scene Root Path` 选项，默认值为 `Assets/Scenes` —— 也就是期望放置该背景演出元素场景资源的目录。你可以修改它（例如为每个演出元素指定独立的文件夹），也可以保持默认。

![](https://i.gyazo.com/0f3c0be40941ad739f2c873c5fbf6e51.png)

::: info NOTE
场景背景的资源（外观）名称必须等于场景文件相对该根目录的路径。例如，如果场景根目录为 `Assets/Scenes`，且你有 `Assets/Scenes/Sphere.unity` 和 `Assets/Scenes/Sub/Cylinder.unity`，那么它们对应的外观名称应为 `Sphere` 与 `Sub/Cylinder`。
:::

在指定的根目录下创建一个新场景（或移动现有场景），并确保场景中至少有一个挂在根对象上的 [Camera](https://docs.unity3d.com/ScriptReference/Camera.html) 组件。当加载场景背景时，Naninovel 会将一个 RenderTexture 分配给场景中找到的第一个摄像机。随后，该 RenderTexture 会被映射到一个背景精灵上，用于在 Naninovel 的场景空间中显示场景背景。这样场景背景便能与其它背景和角色演出元素共存，支持所有背景过渡效果，并根据屏幕比例自动缩放。

确保在世界空间中摆放场景中的物体，使其不会与其他可能被同时加载的场景内容重叠（例如同一 naninovel 脚本中引用多个场景背景时）。另外注意，如果场景背景中的对象靠近全局原点（`x0 y0 z0`），它们可能会被 Naninovel 的主摄像机渲染；为避免此情况，你可以将所有场景物体整体偏移，或者在 `Configuration -> Engine -> Override Objects Layer` 中配置使用 Unity 的 [层（Layer）](https://docs.unity3d.com/Manual/Layers.html) 将 Naninovel 相关物体隔离。

场景设置完成后，通过 `Naninovel -> Configuration -> Backgrounds` 创建一个新的背景演出元素，选择 `SceneBackground` 实现，并将场景资源添加到该演出元素的资源列表中。

当为 Scene Background 分配资源时，关联的场景会自动加入 [Build Settings](https://docs.unity3d.com/Manual/BuildSettings.html)。如果出现提示说场景未加入构建，请手动将其加入。

现在你就可以使用 [@back] 指令控制该场景背景演出元素，例如：

```nani
; 显示 "Scene" 背景演出元素，内容来自 "Sphere" Unity 场景。
@back Sphere id:Scene
; 使用 "RandomCircleReveal" 特效，将该演出元素切换到 "Sub/Cylinder"。
@back Sub/Cylinder.RandomCircleReveal id:Scene
```

::: tip
在使用 Unity 场景组合背景时，建议编写一些 [自定义指令](/zh/guide/custom-commands) 来控制场景状态（例如修改灯光颜色以改变昼夜、移动相机以改变视角），而不是为每种外观都创建一个独立场景。这样在同时加载多个场景时，你就不必再到处追踪物体的位置以避免相互重叠。
:::

::: tip EXAMPLE
关于如何设置场景背景的示例，请参考 [场景背景示例](/zh/guide/samples#scene-background)。
:::

## 渲染到纹理

可以将所有实现类型的角色与背景演出元素（**除了通用以外**）渲染到一个纹理（Texture）资源中，然后将该纹理应用到自定义 UI、文本输出窗、材质或任何其它支持纹理输入的对象上。背景演出元素的渲染到纹理设置方式与角色的几乎相同；详细步骤和示例请参考这里的指南：[角色渲染到纹理](/zh/guide/characters#render-to-texture)。
