**English** | [简体中文](README.zh-CN.md)

# RH Video Replica · 镜作

**Turn the shot functions, pacing, and visual progression of a real ad into an ad for a new product.**

An agent Skill for Codex, WorkBuddy, and DeepSeek Harness: connect reference analysis and storyboard agreements with RunningHub generation, precise brand typography, audio composition, and documented review.

> V2 includes a real worked example. The workflow needs an agent to interpret and review footage. It does not promise frame-exact recreation of arbitrary ads, character motion imitation, or unattended commercial-ready output.

![MELLOW concept coffee ad end card](demo/mellow/poster.jpg)

**[Watch the 15-second MP4](https://github.com/ultimatech-cn/rh-video-replica/raw/refs/heads/main/demo/mellow/final.mp4)** · **[Detailed case · 中文](docs/CASE_STUDY.md)** · **[Download the Skill](https://github.com/ultimatech-cn/rh-video-replica/releases/latest)** · **[Agent instructions · 中文](SKILL.md)**

## What it does

| Input | Workflow | Deliverable |
|---|---|---|
| Local real-world reference ad | Frame extraction, candidate cuts, and visual analysis | Source notes, timestamped evidence, and observations |
| Product images and verified selling points | Retain shot functions while adapting the product and setting | Timed storyboard and acceptance criteria |
| Duration, aspect ratio, generation mode | Upload inputs; submit or resume RunningHub tasks | Task records, costs, and raw generated footage |
| Cloud footage, brand copy, and audio | Edit footage, render accurate typography, and mix sound | H.264/AAC MP4, edit plan, and technical QC |
| Actual output | Agent review and repair by segment | Pass, partial match, or failure with follow-up records |

Scripts handle deterministic execution. The agent handles visual interpretation, adaptation decisions, and content review. Candidate cuts are not confirmed shots; passing technical QC does not establish that the ad meets its creative requirements.

## Two generation modes

| Mode | What the model receives | Best fit | Observed behavior |
|---|---|---|---|
| `reference_video_and_product` | Real ad excerpt + new product image + prompt | Stronger structural retention when the reference can be used | May also retain the original scene and people; review required |
| `storyboard_and_product` | New product image + an agent-written storyboard derived from the reference | New settings, a different brand, and removal of original people | Used for the final demo; the model did not directly receive the reference video |

The recorded mode must match the actual inputs. Do not describe a model as having seen a video it was never given, or an unfinished task as successful.

## Demo: a real NESCAFÉ ad → MELLOW concept coffee

The reference is **NESCAFÉ — The Life of a Coffee Bean**, a 75-second film awarded **Silver in Cinematography: Single at Kancil Awards 2022**. [Original work and production credits](https://kancilawards.com/winners/entry/2022/6539)

Only the **47–62 second process segment** was selected: “bean turning → sampling → establishing the space → operational inspection → grinding transformation → new product end card.” This is a **15-second excerpt adaptation**, not a recreation of the full 75-second film.

- **Round 1:** supply the reference excerpt and product image directly. The result retained more of the structure but also kept an original person's face, so it was not used as the final film.
- **Round 2:** generate new scenes from the adapted storyboard. The final edit uses **11.4 seconds** of this cloud output, followed by a **3.6-second** product end card made from a checked product image, a subtle pullback, exact typography, and original synthesized music.
- **Output:** 1280×720, 16:9, 30 fps, 15 seconds. Full decoding, duration, and black-frame checks passed.
- All six segments have an intended counterpart. The manual/agent review rated three as matching the intent and three as partial matches. Missing steam, the sampling tool's movement, and the grinding texture/machinery realism remain imperfect. This is a single-case review, **not a general recreation success rate**.
- The two RunningHub video generations cost **CNY 12.6 + CNY 6.3 = CNY 18.9**, recorded on November 23, 2026. This excludes agent, image-generation, and other service costs. Live prices follow the service page and billing record.

[demo/mellow](demo/mellow) includes the final video, second-round raw footage, product image, prompts, edit plan, and review JSON. The original ad, its audio, and extracted reference frames are not distributed in this repository.

## Install

Requires **Python 3.10+**. A virtual environment is recommended:

```bash
git clone https://github.com/ultimatech-cn/rh-video-replica.git
cd rh-video-replica
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python scripts/replica.py doctor
```

The workflow prefers system FFmpeg and otherwise uses the version bundled with `imageio-ffmpeg`. On macOS it searches for a Chinese font automatically. On other systems, set `VIDEO_FONT` to the path of an installed Chinese font when rendering Chinese copy.

Import `rh-video-replica-skill-v2.0.0.zip` from the release into your host, or copy the root Skill files into its Skill directory:

| Host | Installation | Recorded status |
|---|---|---|
| Codex | `~/.codex/skills/rh-video-replica/`, containing `SKILL.md` directly | Used for project production and script execution |
| DeepSeek Harness | Project `.dsh/skills/rh-video-replica/` | Directory adaptation prepared; target-session import/triggering not validated |
| WorkBuddy | Skills → add/upload Skill → select the Skill ZIP | Based on official installation guidance; client import/triggering not validated |

Client entry points may change; see [installation notes and official sources · 中文](references/installation.md). Do not use the entire large project directory as a Skill installation package. The agent instructions and detailed references currently remain in Chinese; an English invocation can request English deliverables.

## Ask your agent to use it

```text
Use rh-video-replica. Reply and deliver the documents in English.
Reference ad: /absolute/path/reference.mp4; analyze seconds 12–27.
New product image: /absolute/path/product.png.
Output: 15 seconds, 16:9. Use only the product facts I provide.
Replace the original setting and people using storyboard_and_product.
First provide the source, segment observations, adaptation agreement,
and estimated cost. After generation, deliver the MP4, prompts, edit plan,
actual costs, and content/technical review, identifying unmet requirements.
```

The host needs local image access, Python execution, and file output. Cloud calls also require a locally configured API key. A text-only host cannot complete the full video delivery.

## Rebuild locally without creating a new RunningHub task

```bash
# Validate the sample storyboard timing, scope, and product claims
python scripts/ad_audit.py validate demo/mellow/adaptation-contract.json \
  --out output/contract-validation.json

# Re-edit existing repository footage, typography, and music
python scripts/replica.py render demo/mellow/edit-plan.json --out output/mellow

# Check the final video
python scripts/replica.py qc output/mellow/final.mp4 --expected 15 \
  --out output/mellow/check.json

# Run the 28 local tests without paid API calls
python -m unittest discover -s tests -v
```

Rebuilding the original synthesized audio with `demo/mellow/build_audio.py` additionally requires `numpy`. The demo already includes that audio, so ordinary re-rendering does not need numpy.

## Run your own cloud generation

Read the [RunningHub integration guide · 中文](references/runninghub.md) first. Keys are read only from environment variables. **Do not commit `.env`, task credentials, or asset URLs containing authentication parameters.**

```bash
# Set RUNNINGHUB_API_KEY securely in your own terminal.
# The default service host is https://www.runninghub.cn.

# Creates a NEW paid generation task: confirm inputs, settings, and cost first.
python scripts/replica.py rh-model-run demo/mellow/model-job.json \
  --journal projects/my-ad/task.json

# Resume polling and download only; these do not resubmit the task.
python scripts/replica.py rh-poll projects/my-ad/task.json --timeout 900 --interval 10
python scripts/replica.py rh-download projects/my-ad/task.json --out projects/my-ad/raw
```

Before submitting, the client exclusively creates a task journal. If the submission response is lost, it records `submission_unknown` and stops. If a task ID already exists, resume polling only. Do not delete the journal and blindly resubmit. The current Wan reference route has input-duration constraints that the script checks in advance.

**Live integration coverage:** the two generations were submitted through the official Playground. The final task was queried and downloaded through the official V2 API. Python submission/upload clients have mock-transport tests; a live automatic upload-to-submission chain has not yet been completed. CI needs no key and does not call RunningHub.

## Repository layout

```text
SKILL.md                 Agent instructions
agents/openai.yaml       Skill display metadata
scripts/                 Media processing, adaptation contracts, RH client, CLI
references/              Integration, installation, contracts, provenance
assets/                  Sample edit plans and browser-result handoff helper
experimental/            ComfyUI candidate graphs not imported/run on RH
demo/mellow/             Final video, raw footage, product image, rebuild inputs
docs/CASE_STUDY.md       Real-ad case, two rounds, and observed deviations
tests/                   Local tests without paid calls
```

## Current scope

- No bypassing video-site download restrictions; no character identity, lip-sync, or dance transfer.
- Generated process scenes do not verify a real manufacturing process. A concept product cannot inherit origin, certification, environmental, or sales claims from the reference ad.
- Reference audio is not used by default. Dialogue needs a separately produced and mixed audio track.
- Product lettering, tool movement, hands, and physical processes need review. Important packaging can use a verified product image in a clearly identified end card.
- ComfyUI graphs in `experimental/` have not been imported and run on the platform; they are not part of the demonstrated route.
- There is no separately deployed public product website. GitHub is the source, package, and demo entry point.

## License and provenance

Original code and written documentation use the [MIT License](LICENSE). See [NOTICE.md](NOTICE.md) for generated demo media and third-party brand/service scope, and [references/provenance.md](references/provenance.md) for interface and implementation sources. This project has not acquired commercial rights to the original ad; confirm the appropriate basis for using any real reference.
