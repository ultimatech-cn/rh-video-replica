[English](README.md) | **简体中文**

# 镜作 · RH Video Replica

**把真实广告的镜头功能、节奏与视觉递进，改编为新商品广告。**

一个供 Codex、WorkBuddy、DeepSeek Harness 等工具型 Agent 使用的 Skill：从参考分析、分镜约定、RunningHub 生成，到品牌排版、声音合成与验收记录。

> 当前是有真实样例的 V2 验证版。它需要 Agent 理解与审片，不承诺任意广告逐帧一致、人物动作模仿或无人验收的一键商用。

![MELLOW 概念咖啡广告落版](demo/mellow/poster.jpg)

**[观看 15 秒演示 MP4](https://github.com/ultimatech-cn/rh-video-replica/raw/refs/heads/main/demo/mellow/final.mp4)** · **[详细案例](docs/CASE_STUDY.md)** · **[下载 Skill 安装包](https://github.com/ultimatech-cn/rh-video-replica/releases/latest)** · **[Skill 指令入口](SKILL.md)**

## 能做什么

| 输入 | 处理 | 交付 |
|---|---|---|
| 本地真实参考广告 | 抽帧、候选切点与画面分析 | 来源、时间证据与参考观察 |
| 商品图片与已核实卖点 | 保留镜头功能，改写商品与场景 | 按时间组织的改编分镜与验收条件 |
| 时长、比例、生成方式 | 上传素材、提交/恢复 RH 任务 | 任务记录、费用与原始生成 |
| 云端画面、品牌文案、音轨 | 剪辑、准确排字、合成声音 | H.264/AAC MP4、编辑计划、技术 QC |
| 实际输出 | Agent 逐段审片与修复 | 通过、部分符合、失败及处理记录 |

脚本负责确定性执行；Agent 负责视觉理解、改编决策和内容验收。候选切点不是已确认镜头，技术 QC 通过也不代表广告内容合格。

## 两种生成方式

| 模式 | 模型收到什么 | 适合什么 | 实测发现 |
|---|---|---|---|
| `reference_video_and_product` | 真实广告片段 + 新商品图 + 提示词 | 有相应使用依据、希望较强保留参考结构 | 可能连原场景与人物也保留，必须审片 |
| `storyboard_and_product` | 新商品图 + Agent 根据参考改写的分镜 | 需要新场景、换品牌和去除原人物 | 本案例最终采用；模型没有直接接收原视频 |

模式必须显式对应输入。不会把“模型未收到原视频”写成“模型看过原片”，也不会把未完成的任务称为成功。

## 演示：真实雀巢广告 → MELLOW 概念咖啡

参考是 **NESCAFÉ — The Life of a Coffee Bean**，Kancil Awards 2022 **Cinematography: Single 银奖**，原片 75 秒。[作品与制作信息来源](https://kancilawards.com/winners/entry/2022/6539)

本次只选 **47–62 秒工艺段**，拆为“翻豆 → 取样 → 空间建立 → 操作检查 → 研磨转化 → 新商品落版”。这是 15 秒选段改编，**没有完整复刻 75 秒原片**。

- 第一轮直接给模型参考视频和商品图：结构保留较强，但留下原人物脸，未作为最终成片。
- 第二轮根据拆解分镜生成新场景：前 **11.4 秒**使用这次云端视频；后 **3.6 秒**用核对过的商品主视觉做轻微拉远、准确排字，再加入原创合成配乐。
- 输出 **1280×720、16:9、30fps、15 秒**，完整解码、时长、黑场检查通过。
- 六段均有内容对应；人工/Agent 判定三段符合意图、三段部分符合。蒸汽缺失、取样器具的驱动关系、研磨颗粒与实机感仍有偏差。这是单例评审，**不是泛化“复刻率”**。
- 本例两轮 RH 视频实际费用 **¥12.6 + ¥6.3 = ¥18.9**，记录于 2026-11-23。当前价格以服务页面与账单为准，不含 Agent、图片生成或其他服务成本。

演示素材见 [demo/mellow](demo/mellow)，包括成片、第二轮原始生成、商品图、提示词、编辑计划和验收 JSON。原广告、原音轨和原片抽帧不随仓库发布。

## 安装

Python **3.10+**。推荐使用虚拟环境：

```bash
git clone https://github.com/ultimatech-cn/rh-video-replica.git
cd rh-video-replica
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python scripts/replica.py doctor
```

有系统 FFmpeg 时优先使用，否则使用 `imageio-ffmpeg` 自带版本。中文排字在 macOS 自动寻找字体；其他系统请设置 `VIDEO_FONT` 为已安装中文字体的路径。

将 Release 中的 `rh-video-replica-skill-v2.0.0.zip` 导入宿主，或把根目录下的 Skill 文件复制到相应目录：

| 宿主 | 安装方式 | 验证范围 |
|---|---|---|
| Codex | `~/.codex/skills/rh-video-replica/`，目录下直接有 `SKILL.md` | 本项目制作与脚本执行使用 Codex |
| DeepSeek Harness | 项目 `.dsh/skills/rh-video-replica/` | 目录适配已准备，未做目标会话导入触发验收 |
| WorkBuddy | 技能 → 添加/上传技能 → 选择 Skill ZIP | 参考官方安装说明，未做客户端导入触发验收 |

具体入口可能随客户端版本调整，见 [安装说明与官方来源](references/installation.md)。不要把整个大型项目目录当作 Skill 安装包。

## 怎样让 Agent 使用

```text
使用 rh-video-replica。
参考广告：/absolute/path/reference.mp4，分析其中 12–27 秒。
新商品图：/absolute/path/product.png。
输出：15 秒、16:9；卖点仅使用我提供的事实。
希望更换原场景与人物，采用 storyboard_and_product。
先给出来源、逐段观察、改编约定和费用预估；生成后交付 MP4、
提示词、编辑计划、实际费用及内容/技术验收，明确未通过项。
```

宿主需要查看本地图片、执行 Python 和保存文件；要自行调用云端，还需要本机配置 API Key。仅有聊天能力的宿主不能完成整个视频交付。

## 本地先跑一遍，不产生 RH 新任务

```bash
# 验证示例分镜的时间、范围和商品主张
python scripts/ad_audit.py validate demo/mellow/adaptation-contract.json \
  --out output/contract-validation.json

# 用仓库内已有生成素材重新剪辑、排字、配乐
python scripts/replica.py render demo/mellow/edit-plan.json --out output/mellow

# 检查交付成片
python scripts/replica.py qc output/mellow/final.mp4 --expected 15 \
  --out output/mellow/check.json

# 28 项本地测试，不调用收费 API
python -m unittest discover -s tests -v
```

重建原创合成音轨另需 `numpy`，运行 `demo/mellow/build_audio.py`。音轨已随演示提供，普通复合成无需安装 numpy。

## 运行自己的云端生成

先阅读 [RunningHub 接入说明](references/runninghub.md)。密钥只从环境变量读取，**不要提交 `.env`、任务凭据或带认证参数的素材 URL**。

```bash
# 在自己的终端中安全设置 RUNNINGHUB_API_KEY
# 默认站点为 https://www.runninghub.cn

# 下面会创建一个新的收费生成任务；先确认素材、设置与费用
python scripts/replica.py rh-model-run demo/mellow/model-job.json \
  --journal projects/my-ad/task.json

# 以下只恢复查询和下载，不重新提交
python scripts/replica.py rh-poll projects/my-ad/task.json --timeout 900 --interval 10
python scripts/replica.py rh-download projects/my-ad/task.json --out projects/my-ad/raw
```

提交前独占创建 journal，响应丢失时记录 `submission_unknown` 并停止；已有 taskId 只恢复查询。不要删除 journal 后盲目重提。当前万相参考路线有输入时长约束，脚本会提前检查。

**实网验证范围：** 两轮生成通过官方 Playground 提交；最终任务通过官方 V2 API 查询并下载。Python 提交/上传客户端有模拟传输测试，尚未完成真实自动提交全链路测试。CI 不需要密钥，也不会调用 RH。

## 项目结构

```text
SKILL.md                 Agent 指令
agents/openai.yaml       Skill 展示信息
scripts/                 媒体处理、改编约定、RH 客户端与命令入口
references/              接入、安装、约定和来源说明
assets/                  示例编辑计划与网页结果接回工具
experimental/            尚未在 RH 导入运行的 ComfyUI 候选图
demo/mellow/             成片、原始生成、主视觉和复跑材料
docs/CASE_STUDY.md       真实广告案例、两轮实验与偏差
tests/                   无收费调用的本地测试
```

## 当前限制

- 不处理任意视频网站的下载绕过，不提供人物身份/口型/舞蹈迁移能力。
- 生成场景不证明真实生产流程；概念商品不能借用参考广告的产地、认证、环保或销量主张。
- 参考音轨默认不使用；有对白要求时，需要单独制作并混好音轨。
- 商品字样、器具运动、手部和物理过程仍需审查；重要包装可用已确认图片做明确标记的落版。
- ComfyUI 候选图位于 `experimental/`，未在平台导入实跑，不属于本次已验证路线。
- 当前没有公开部署的独立产品网站。GitHub 是源码、安装包和演示入口。

## 许可证与来源

原创代码和文字文档使用 [MIT License](LICENSE)。生成演示素材与第三方品牌/服务的适用范围见 [NOTICE.md](NOTICE.md)，接口参考与实现来源见 [references/provenance.md](references/provenance.md)。本项目没有取得原广告的商业素材授权，使用真实参考时请自行确认相应使用依据。
