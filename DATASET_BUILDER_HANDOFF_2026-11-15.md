# TreeWeaver 合成树数据集制作交接说明

交接日期：2026年11月15日。对象：接手数据集建设的小组成员。当前任务是完善现有资产与生成管线，先交付可以复现和逐张验收的小批样例，再扩充正式训练集。

研究目标是相邻树的实例分离与像素归属，以及目标树被遮挡后的树冠补全。主干与树冠部件标签、米制深度和林木参数为后续结构分析与测量提供依据。本次交接不要求你先训练模型。

## 一 项目链接和文件关系

- 公开仓库：https://github.com/SliverBronya/TreeWeaver
- 当前远端进度：https://github.com/SliverBronya/TreeWeaver/blob/main/PROGRESS.md
- 当前远端要求：https://github.com/SliverBronya/TreeWeaver/blob/main/DATASET_REQUIREMENTS.md
- 本次制作源码更新：https://github.com/SliverBronya/TreeWeaver/blob/main/TreeWeaver-dataset-builder-source-20261115-v1.zip
- 历史源码与样例：https://github.com/SliverBronya/TreeWeaver/blob/main/TreeWeaver-metric-20261115-v1.zip
- 历史深度与EXR：https://github.com/SliverBronya/TreeWeaver/blob/main/TreeWeaver-metric-depth-20261115-v1.zip

负责人另发的 TreeWeaver-dataset-builder-20261115-v1.zip 是本次完整制作交接包，包含本地新增的路径兼容修正、运行入口和资产。使用这个包作为本次工作起点，不需要再叠加解压上述两个历史包。GitHub 上已有源码以 ZIP 快照为主，不能假设 git clone 后就有展开的项目源码与大型资产。

上面的本次制作源码更新是小型源码包，不含29棵树等原始资产，也不替代负责人另发的完整制作交接包。后续协作优先按本说明和完整交接包的版本开展。

交接包内有展开的项目源码、29棵fig GLB、7份地面或天空环境素材、当前场景的RGB与单目标对照、V/A/部件标签、米制NPZ和原始EXR、严格检查失败报告及联合有效域通过报告。根目录的 HANDOFF_MANIFEST.json 记录每个文件的相对路径、大小和SHA256。

不含 Blender 可执行程序、Python 运行时、模型权重、全部外部数据集、20260411手机原始照片及其他大型树资产。仅复现当前场景不需要这些外部数据或模型。29棵fig是同类资产变体，不代表29个树种。

## 二 当前样例和已知问题

当前样例路径为 reports/cuda-metric-scene-2026-11-15-03/images/000001.png；同相机移走邻树的目标对照在 counterfactuals/000001/target_1_rgb.png。根目录 previews/labels-and-depth.png 对照展示RGB、单目标图、visible、amodal、hidden、归属、部件及深度。这里展示的是渲染真值，不是模型预测。

现有场景实际用RTX4070 Laptop的CUDA渲染，1024×768，RGB为64 samples。主目标完整投影高度占画面81.51%，主目标已知域隐藏率7.87%；这个场景的主要外部遮挡来自另一棵树。主树完整入镜，右侧邻树截断，因此邻树不能直接作为完整树补全或完整尺寸监督。背景是3棵共享网格远树，地面、背景层次和树种变化仍明显不足。

RGB-D联合已知域保留原RGB分割已知树像素的99.03%。原始已知树像素334793个，其中1753个缺有效深度；另外还有首采样对象ID和多数覆盖分割不一致的像素。联合有效域有效树像素331548个。两个检查报告都保留，不通过插值或填0伪造完整深度。

当前标签中的 stem 主要来自源bark材质，包含木质枝条，尚未认证为植物学主干。9m是人为指定的虚拟顶点包络尺度；DBH和木质材积仍为null。原始树皮网格有未闭合或接缝问题，不能直接计算体积当真值。当前 training_allowed=false；部分合同通过不表示可直接正式训练。

## 三 数据必须达到的要求

1. 主目标大且完整。完整amodal包围盒高度占原生图像高度75%至95%，完整保留根部、树梢与左右边缘。宽冠优先完整入镜并另记构图层。不用实心像素占比代替包围盒高度，也不填叶隙。
2. 主遮挡批的树间真实遮挡率为5%至25%，分5%至15%和15%至25%两层；保留无遮挡和接触对照。目标前后方向都要有。遮挡比例按像素计算，不能用框重叠或树间距离代替。
3. 环境接近真实照片。有地面与接触阴影、天空、远景和中低空背景；逐步加入低矮植被、杆件或围栏等有实际几何的遮挡物。先固定资产改善场景，再增加多样性，避免同时改动所有变量。
4. 原生坐标对应。RGB、V/A、部件、owner、known、深度和相机参数要对应同一场景与相机。当前是1024×768，不能把掩码直接拉成方形。天空和无命中深度必须无效。
5. 每棵树参数有依据。树高、冠幅X/Y、DBH和木质体积记录value、unit、status、provenance、definition及不可用原因。不可用填null，不填0；虚拟几何真值与现场实测值明确区分。
6. 可复现且无泄漏。记录资产hash、源族、场景种子、相机、物体变换、灯光、生成配置与代码版本。主树、邻树、远景树、变体及同机位控制图的来源一起分组；同族不能跨train、val、test。29个变体是否同源需要保守核查。

20260411仅作为构图和可观察遮挡类型的开发参考，不用于训练或训练背景，不从RGB猜隐藏真值。目前未附这批私人照片，需要负责人另外提供观察统计或在本机讨论。

## 四 标签和计算方式

V是全场景目标实际可见投影；A是在同一机位移走规定外部遮挡物后的该树投影；Ω是双方共有已知域。树自身叶片遮住树干仍保留，不能移走叶片再把木质投影叫visible_stem。amodal恢复外部遮挡下的该树投影，不推断树背面不可见的三维结构，也不填平树冠中的真实叶隙。

隐藏区域 H = A 与 非V 与 Ω 的交集；总隐藏率 = H的像素数 / A与Ω交集的像素数。树间隐藏率的分子进一步限制为实际邻树遮挡的像素，其他灌木、杆件等单独统计。当前 result.json 中的 target_hidden_fraction 来自原RGB分割已知域；资格报告可能在联合有效域重算，必须同时报告所用域与分母。

owner是单像素最前方实例ID；known=false为未知，不当背景训练。保留unknown和透明边缘，不使用整棵树的全局前后序减掩码来伪造局部可见关系。灌木、杆件、围栏与其他树分别记录ID、类别、遮挡mask和局部前后证据。

部件应最终输出main_stem、crown和part_unknown及valid，互斥且覆盖允许监督的可见支持。先核查主干结构，再从源bark拆出主干与其他枝条。真实多干和低位分叉必须单列，不能靠最大连通域删除成单干。目标树冠边界约束采用原图绝对左右边界及逐行valid，只作软约束，保留天然凹陷和叶隙；当前米制试跑未提供完整冠形监督，需补出并检查。

metric_depth.npz 的 depth_z_m 是相机Z方向深度，depth_ray_m 是射线距离，单位m；valid标明有效命中。object_id、world_position_m、K及相机位姿一并保留。像素中心为u+0.5、v+0.5，相机坐标为右、下、前。灰度或彩色深度预览不能用于测量。

DBH需先确认主干和地表基点，再按离地1.3m截面协议计算；倾斜、多干、坡地和低位分叉另立协议。木质材积要求明确统计口径、封闭无自交且不重复计积的木质实体；凸包、冠体积或外观树内藏一个代理圆柱都不能充木质体积GT。修复副本必须参与实际RGB渲染并重新核验。

## 五 在另一台电脑复现

当前快捷入口面向Windows和NVIDIA CUDA。建议先用Blender 4.2.3以复现已有结果；Python用3.11或3.12。只有CPU或其他GPU的电脑可先做资产修复、规范和静态审计，当前脚本不会回退到CPU渲染。

1. 解压到任意工作目录，例如 E:\TreeWeaver。进入包含 pyproject.toml、scripts、workers、tools 的项目根目录。不要把tools另拆到其他位置。本次已把生成器资产路径改成随项目定位。
2. 安装Python，准备Blender。Blender不在包里，可使用官方4.2.3 Windows便携包：https://download.blender.org/release/Blender4.2/blender-4.2.3-windows-x64.zip 。解压后记住blender.exe完整路径。无需安装Torch或下载SAM权重。
3. 用PowerShell逐行运行下面命令。Python建立环境；默认不需要激活环境，也不需要修改系统执行策略。渲染脚本不会修改驱动、电源或TDR，也不会重启机器。

```powershell
python scripts/check_dataset_handoff.py
python -m venv .venv-dataset
.\.venv-dataset\Scripts\python.exe -m pip install -r requirements-dataset.txt
.\.venv-dataset\Scripts\python.exe -m pytest tests/test_metric_camera.py tests/test_metric_dataset_contract.py tests/test_metric_asset_audit.py tests/test_qualify_metric_scene_cli.py -q
```

校验应为failures空列表。手动渲染请把下面的Blender路径替换成你本机路径，并始终使用新输出目录；这些命令不会覆盖交接样例。

```powershell
.\.venv-dataset\Scripts\python.exe scripts/run_cuda_tree_render.py --blender "E:\Apps\blender-4.2.3-windows-x64\blender.exe" --out reports/team-repro-001
.\.venv-dataset\Scripts\python.exe scripts/prepare_rendered_metric_scene.py --render reports/team-repro-001 --joint-depth --output reports/team-repro-001-joint
.\.venv-dataset\Scripts\python.exe scripts/qualify_metric_scene.py --scene reports/team-repro-001-joint/scene.json --labels reports/team-repro-001-joint/labels.npz --rgb reports/team-repro-001/images/000001.png --focal-id 1 --occlusion-scope tree --allow-incomplete-metrics --output reports/team-repro-001-check
```

也可用 .\scripts\run_dataset_handoff.ps1 -Blender "你的blender.exe路径" -OutputName team-repro-001 一次完成上述三步。若系统阻止PS脚本，直接用前述Python命令即可，不需修改系统策略。

检查qualification_passed、issues和training_allowed，不只看进程退出码。--allow-incomplete-metrics明确只检查部分合同，不能据此声称完整物理真值合格。实际叶缘采样和GPU平台差异可能使新运行不通过，应保留失败报告再修，不能直接复制旧报告。

## 六 按这个顺序推进

第一步先复现一张。交回RGB、8格标签预览、GPU日志与watchdog、scene.json、NPZ、qualification报告。负责人确认取景、标签和深度后再改场景。

第二步固定1830与430这对树、相机和光照，只改邻树相对位置，先交无遮挡、接触、5%至15%、15%至25%各一组。每次计算实际树间隐藏率，不能只改位置后目测。render_cuda_tree_scene.py目前是固定双树资格样例，还没有可直接运行的多配置批生成器；需把资产、变换、相机、环境、种子抽成显式配置，生成时保存配置，不要直接跑旧generate_fig_500.py生成一批就当新米制数据。

第三步处理透明边缘的共同采样和有效域。对RGB、实例ID、部件ID、深度命中定义进行对照，保留原始分割known与RGB-D联合known，分别报告覆盖率。不能缩小有效域后只报通过，也不能把插值当渲染真值。主干语义与木质修复可在资产副本上同步推进，但应分别交付证据。

第四步改善地面、远景与中低空层次；固定主树与遮挡配置进行前后对照。7份环境素材都在tools/assets/env，但当前脚本只用其中3份，其余用于受控变更。新增灌木等资产需另登记来源、许可、尺度与语义，当前包不含专门灌木或围栏模型。

第五步交一个建议的16场景试制批，作为审查批而非既定训练集规模。四类各4个场景，覆盖至少4组资产组合、两个目标前后方向；每组保存同机位单目标控制。接触类不强求5%遮挡，clear与contact不要硬套occluded_primary的门槛。现有资格CLI只有occluded_primary、clear_control与stress三档，contact需先人工单列并补协议/检查后再正式准入。

第六步完成资产来源分组、去重和逐图审查，冻结清单与任务级valid后，再讨论训练量和数据划分。单一源族不足以建立独立跨资产测试时如实说明，不能把不同相机图随机切分当独立泛化。

## 七 改哪些文件和怎样交回

- scripts/render_cuda_tree_scene.py：CUDA、场景布局、背景、取景和RGB渲染；当前固定样例不是批生产器。
- scripts/generate_fig_500.py 与 scripts/fig_data_protocol.py：资产加载、同机位控制、保留alpha的实例与部件ID机制。新场景复用这些机制，保留历史入口。
- scripts/blender_metric_export.py：Position、Z深度、对象ID、相机和原始EXR导出。
- scripts/prepare_rendered_metric_scene.py：产物转统一标签和scene元数据；当前仍固定fig资产位置及9m声明，推广配置时必须同步修正，不能给任意树误写9m或固定截断标志。
- scripts/qualify_metric_scene.py 与 workers/tree_models/metric_dataset_contract.py：合同和资格检查；不放宽阈值来掩盖标签缺陷。
- scripts/audit_metric_tree_assets.py 与 workers/tree_models/synthetic_geometry.py：几何审计及有条件的物理参数计算。
- configs/research/metric-tree-dataset-2026-11-14.json：生产要求；docs/collaboration/PROGRESS.md记录实际进度与失败。

先用独立副本目录工作，输出保存到新reports目录；不覆盖历史样例或改原始GLB。建议分支名 codex/dataset-成员名。公开仓库可读不代表有写权限，需要负责人授予协作者权限后再推送，或用fork提PR。远端目前是快照式交接，第一次源码入库先协商展开目录，不要用本地独立历史强推覆盖main。

如果暂时不使用Git，交回dataset-update-成员名-日期.zip，至少包含变更源码与配置、资产修复副本及其hash、16场景索引、所有样例的RGB/V/A/owner/parts/known/depth和相机、原始EXR、日志与检查报告，以及CHANGELOG.md。资产未变可只写原hash，不必重复传整套29棵树。

CHANGELOG.md写明本次改动、输入版本、实际命令、成功和失败场景数、每张主树取景与树间遮挡率、RGB-D覆盖率、无效像素原因、物理参数可用状态、尚未解决项和下一步。每张记录scene_id、源族、种子、配置路径和产物hash。先交一个合格复现样例让负责人确认，再做16场景试制批。

## 八 数据交付后用于什么

可见整树V用于LoRA/PF、正负提示和漏分检验；owner与局部前后证据用于实例分离、降低相邻树污染并同时保召回。合格可见部件用于主干与树冠输出，unknown不参与强监督。A、H和可见率用于amodal与visibility训练，无遮挡对照约束不过补；树冠曲线在同坐标有效边界上作软约束。深度、相机与已核验树木参数用于几何分析与独立测量评价，不默认变成RGB模型的输入。

训练顺序为可见LoRA/PF、部件与ownership、amodal与visibility、curve与occluder及局部层次。固定数据划分、提示预算、基础权重和评估协议，每次只改一项。训练由模型负责人安排，数据成员不能仅因脚本运行成功就把training_allowed改为true。
