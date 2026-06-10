# RunPod에서 ComfyUI 실행하기 가이드

> 처음 하는 사람도 따라할 수 있게 썼어. 차례대로만 하면 돼.

---

## 🗺️ 전체 흐름 한눈에 보기

```
1. RunPod에서 GPU 빌려서 켜기
2. 내 SSH 키 등록하기
3. ComfyUI 설치하기
4. 필요한 커스텀 노드 설치하기
5. 필요한 모델 파일 다운로드하기
6. ComfyUI 실행하고 접속하기
7. 워크플로우 불러와서 실행하기
```

---

## 1단계. RunPod에서 포드 만들기

1. [runpod.io](https://runpod.io) 에 로그인
2. 왼쪽 메뉴 **Pods** → 오른쪽 위 **+ GPU Pod** 클릭
3. GPU 선택 (이 가이드 기준: **RTX PRO 4500 Blackwell** 또는 비슷한 것)
4. 템플릿: **RunPod Pytorch** 선택
5. **Deploy** 클릭해서 포드 생성

> 포드가 켜지면 초록 불이 들어와. 몇 분 걸릴 수 있어.

---

## 2단계. 내 컴퓨터 SSH 키 확인하기

SSH 키가 있어야 RunPod에 안전하게 접속할 수 있어.

터미널(맥: 터미널 앱) 열고 아래 명령어 실행:

```bash
cat ~/.ssh/id_ed25519.pub
```

이렇게 생긴 긴 문자열이 나와야 해:
```
ssh-ed25519 AAAAC3... cm.kafa.01@gmail.com
```

> 아무것도 안 나오면 `ssh-keygen -t ed25519` 실행해서 키 먼저 만들어.

---

## 3단계. RunPod에 SSH 키 등록하기

1. RunPod 포드 페이지에서 포드 클릭 → **Connect** 탭
2. JupyterLab 링크 클릭해서 열기
3. JupyterLab 안에서 터미널 열기 (상단 **+** → Terminal)
4. 아래 명령어 실행 (이메일 부분을 본인 이메일로 바꿔):

```bash
mkdir -p ~/.ssh && echo "여기에_2단계에서_복사한_내_공개키_붙여넣기" > ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
```

> ⚠️ 공개키는 2단계 `cat ~/.ssh/id_ed25519.pub` 로 나온 **본인 키 그대로** 써야 해. (예시 키를 그대로 붙이면 접속 안 됨)
> 덮어쓰기(`>`)를 쓰면 이전에 잘못 넣은 키가 있어도 깨끗하게 정리돼.

### 💡 SSH 키를 매번 등록하기 싫으면 (강력 추천)

RunPod **계정 Settings → SSH Public Keys** 에 위 공개키를 한 번 등록해두면,
이후 포드를 Start할 때마다 **자동으로 주입**돼서 이 3단계를 다시 안 해도 돼.
(아래 "다시 시작할 때" 참고 — Stop/Start 하면 `/root` 가 초기화돼서 수동 등록한 키는 매번 날아가거든.)

---

## 4단계. SSH 접속 정보 확인하고 접속하기

RunPod **Connect** 탭에 두 가지 접속 방법이 나와:

```
SSH:              ssh j515...@ssh.runpod.io -i ~/.ssh/id_ed25519
SSH over TCP:     ssh root@213.xxx.xxx.xx -p 36190 -i ~/.ssh/id_ed25519
```

> **TCP 방식 추천** (파일 전송도 가능해)

로컬 터미널에서 접속 테스트:
```bash
ssh root@213.xxx.xxx.xx -p 포트번호 -i ~/.ssh/id_ed25519 "echo 접속성공"
```

`접속성공` 이 나오면 OK!

---

## 5단계. ComfyUI 설치하기

SSH로 접속한 상태에서, 또는 JupyterLab 터미널에서 실행:

```bash
cd /workspace
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
pip3 install -r requirements.txt --break-system-packages
```

설치에 2~3분 걸려. 마지막에 `Successfully installed ...` 뜨면 완료.

---

## 6단계. 필요한 커스텀 노드 설치하기

AI-RENDERER 2.0 워크플로우에 필요한 노드들이야.

```bash
cd /workspace/ComfyUI/custom_nodes

git clone https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite.git
git clone https://github.com/kijai/ComfyUI-KJNodes.git
git clone https://github.com/rgthree/rgthree-comfy.git
git clone https://github.com/jamesWalker55/comfyui-various.git
git clone https://github.com/cubiq/comfyui_essentials.git
git clone https://github.com/yolain/ComfyUI-Easy-Use.git
git clone https://github.com/ltdrdata/ComfyUI-Impact-Pack.git
git clone https://github.com/kijai/ComfyUI-WanVideoWrapper.git
git clone https://github.com/Mickmumpitz/comfyui-mickmumpitz-nodes.git
git clone https://github.com/drozbay/ComfyUI-WanVaceAdvanced.git
git clone https://github.com/pythongosssss/ComfyUI-Custom-Scripts.git
```

각 노드의 의존성 설치:

```bash
pip3 install -r ComfyUI-VideoHelperSuite/requirements.txt --break-system-packages -q
pip3 install -r ComfyUI-KJNodes/requirements.txt --break-system-packages -q
pip3 install -r comfyui_essentials/requirements.txt --break-system-packages -q
pip3 install -r ComfyUI-Impact-Pack/requirements.txt --break-system-packages -q
pip3 install -r ComfyUI-WanVideoWrapper/requirements.txt --break-system-packages -q
pip3 install -r comfyui-mickmumpitz-nodes/requirements.txt --break-system-packages -q
pip3 install -r ComfyUI-WanVaceAdvanced/requirements.txt --break-system-packages -q
pip3 install soundfile gitpython --break-system-packages -q
```

requirements 파일에 안 잡히는 모듈이 있어서 아래도 같이 깔아줘 (안 깔면 노드들이 "누락"으로 떠).
**새로 만든 빈 팟은 이 패키지들이 거의 다 비어 있어서** 한 번에 깔아두는 게 편해:

```bash
# cv2(opencv)      → VideoHelperSuite·Easy-Use·Impact-Pack·Mickmumpitz
# accelerate       → WanVideoWrapper
# soundfile        → comfyui-various
# pywavelets(pywt) → RES4LYF
# scikit-image     → Impact-Pack
# gguf, ftfy       → WanVideoWrapper
# matplotlib       → RES4LYF
# segment-anything → WanVaceAdvanced(Impact 연동)
# piexif           → Impact-Pack
pip3 install opencv-python accelerate soundfile pywavelets scikit-image \
  gguf matplotlib segment-anything piexif ftfy gitpython --break-system-packages -q
```

> 💡 ComfyUI **코어**도 새 팟에선 `sqlalchemy` `alembic` `tqdm` `blake3` 가 빠져 코어 자체가 안 뜨는 경우가 있어
> (`ModuleNotFoundError: No module named 'sqlalchemy'` / `'tqdm'`). 위 5단계 `pip3 install -r requirements.txt` 가
> 끝까지 돌았는지 확인하고, 그래도 나면 `pip3 install sqlalchemy alembic tqdm blake3 --break-system-packages -q` 로 보강해.

---

### ⚠️ 6단계 직후 꼭 확인: torch ↔ torchaudio 버전 맞추기

위 requirements 설치 과정에서 `torchaudio`가 옛날 버전으로 덮여 **torch와 버전이 어긋나는** 일이 생겨.
그러면 ComfyUI 코어가 아예 안 뜨고 이런 에러가 나:

```
OSError: .../libtorchaudio.so: undefined symbol: ...
```

또는 KJNodes·WanVideoWrapper 가 "누락"으로 떠. 먼저 현재 버전을 확인해:

```bash
python3 -c "import torch; print('torch', torch.__version__)"
pip3 show torchaudio | grep Version
```

`torch`와 `torchaudio`의 버전·CUDA(cu130 등)가 **다르면** 셋을 같은 버전으로 통일해 (torch 버전에 맞춰 숫자만 바꿔):

```bash
# 예: torch 2.11.0+cu130 기준. torch가 2.12면 채널을 cu130 그대로 두고 버전만 맞추면 돼.
pip3 install torch==2.11.0 torchvision==0.26.0 torchaudio==2.11.0 \
  --index-url https://download.pytorch.org/whl/cu130 --break-system-packages
```

확인:

```bash
python3 -c "import torch, torchaudio; print(torch.__version__, torchaudio.__version__, torch.cuda.is_available())"
```

세 값의 버전이 같고 마지막이 `True`면 OK.

---

### ⚠️ WanVideoWrapper 가 `cached_download` 에러로 누락될 때

WanVideoWrapper 가 이런 에러로 IMPORT FAILED 되는 경우가 있어:

```
cannot import name 'cached_download' from 'huggingface_hub'
```

구버전 `diffusers` 가 최신 `huggingface_hub` 와 안 맞아서 나는 충돌이야. diffusers 를 최신으로 올리면 해결돼:

```bash
pip3 install -U diffusers --break-system-packages -q
```

> WanVideoWrapper 는 메인 워크플로우 핵심 노드라 이건 꼭 떠야 해.
> (`onnx` `sageattention` 누락 경고는 선택 기능이라 무시해도 됨.)

---

### ⚠️ ffmpeg 설치 (영상 출력 노드용)

`VHS_VideoCombine` 같은 영상 합치기 노드는 **ffmpeg** 이 있어야 해. 없으면 실행 중 이런 에러가 나:

```
ProcessLookupError: ffmpeg is required for video outputs and could not be found.
```

apt 로 설치하고 ComfyUI 를 **재시작**하면 인식돼 (ffmpeg 은 시작할 때 한 번 잡힘):

```bash
apt-get update && apt-get install -y ffmpeg
pip3 install imageio-ffmpeg --break-system-packages -q
```

---

## 7단계. AI 모델 파일 다운로드하기

> ⚠️ 파일이 커서 시간이 꽤 걸려 (총 약 20GB). 기다려야 해.

```bash
mkdir -p /workspace/ComfyUI/models/diffusion_models/wan
mkdir -p /workspace/ComfyUI/models/text_encoders
mkdir -p /workspace/ComfyUI/models/vae
mkdir -p /workspace/ComfyUI/models/loras

# 메인 모델 (~17GB, 제일 큼)
wget -O /workspace/ComfyUI/models/diffusion_models/wan/wan-14B_vace_skyreels_v3_R2V_e4m3fn_v1.safetensors \
  'https://huggingface.co/Inner-Reflections/VACE_Skyreels_V3_R2V_Merge/resolve/main/wan-14B_vace_skyreels_v3_R2V_e4m3fn_v1.safetensors' &

# 텍스트 인코더 (~3.3GB)
wget -O /workspace/ComfyUI/models/text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors \
  'https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors' &

# VAE (~243MB)
wget -O /workspace/ComfyUI/models/vae/wan_2.1_vae.safetensors \
  'https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/vae/wan_2.1_vae.safetensors' &

# LoRA (~303MB)
wget -O /workspace/ComfyUI/models/loras/Wan2.1_T2V_14B_FusionX_LoRA.safetensors \
  'https://huggingface.co/vrgamedevgirl84/Wan14BT2VFusioniX/resolve/main/FusionX_LoRa/Wan2.1_T2V_14B_FusionX_LoRA.safetensors' &

wait
echo "모든 다운로드 완료!"
```

다운로드 완료 확인:
```bash
ls -lh /workspace/ComfyUI/models/diffusion_models/wan/
ls -lh /workspace/ComfyUI/models/text_encoders/
ls -lh /workspace/ComfyUI/models/vae/
ls -lh /workspace/ComfyUI/models/loras/
```

---

## 8단계. ComfyUI 실행하기

새로 만든 포드에는 `tmux`가 없을 수 있어. `tmux: command not found` 뜨면 먼저 설치:

```bash
apt-get update && apt-get install -y tmux
```

tmux를 써야 SSH 끊겨도 ComfyUI가 계속 살아있어:

```bash
tmux new-session -d -s comfy
tmux send-keys -t comfy 'cd /workspace/ComfyUI && python3 main.py --listen 0.0.0.0 --port 8188 2>&1 | tee /workspace/comfyui.log' Enter
```

> ⚠️ `send-keys` 줄 **맨 끝의 `Enter`** 를 빠뜨리면 명령어만 입력되고 실행이 안 돼. 꼭 붙여.
> 안에서 뭐가 도는지 직접 보려면 `tmux attach -t comfy` (빠져나올 땐 `Ctrl+B` 떼고 `D`. `Ctrl+C` 누르면 ComfyUI 꺼지니 주의).

실행됐는지 확인:
```bash
# 서버 뜰 때까지 대기 (30초~1분)
until curl -s http://localhost:8188/system_stats > /dev/null 2>&1; do sleep 3; done && echo "ComfyUI 준비됨!"
```

---

## 9단계. 브라우저로 접속하기

RunPod **Connect** 탭 → **Port 8188 → Jupyter Lab** 옆에 있는 링크 클릭

또는 주소창에 직접:
```
https://[포드ID]-8188.proxy.runpod.net/
```

포드 ID는 RunPod 대시보드에서 확인할 수 있어 (예: `j515lhdqjg16k0`).

---

## 10단계. 워크플로우 파일 업로드하고 실행하기

### 방법 1: 터미널로 업로드 (맥 로컬에서)

```bash
# 워크플로우 폴더 만들기
ssh root@213.xxx.xxx.xx -p 포트번호 -i ~/.ssh/id_ed25519 \
  "mkdir -p /workspace/ComfyUI/user/default/workflows/"

# 파일 전송
base64 -i "워크플로우파일.json" | \
  ssh root@213.xxx.xxx.xx -p 포트번호 -i ~/.ssh/id_ed25519 \
  "base64 -d > /workspace/ComfyUI/user/default/workflows/워크플로우파일.json"
```

### 방법 2: JupyterLab에서 업로드

JupyterLab 왼쪽 파일탐색기 → `/workspace/ComfyUI/user/default/workflows/` 로 이동 → 업로드 버튼 클릭

### 워크플로우 불러오기

ComfyUI 상단 **`+`** 버튼 클릭 → 파일 선택 → 업로드한 JSON 파일 선택

---

## 🔄 다음에 다시 시작할 때 (Stop → Start 후)

> ⚠️ **꼭 알아둬:** RunPod은 포드를 **Stop/Start 하면 `/workspace` 밖이 전부 초기화돼.**
> 즉 다음 두 가지가 매번 날아가:
> 1. **SSH 키** (`/root/.ssh/authorized_keys`) → 내 키로 접속 안 됨 (`Permission denied (publickey)`)
> 2. **pip 패키지 전체** (torch 정렬·cv2·accelerate·코어 의존성…) → ComfyUI 안 뜨거나 노드 누락
>
> `/workspace` 안(노드 폴더·모델·워크플로우)만 살아남아. 그래서 아래 2단계가 필요해.

### 1. SSH 키 다시 등록 (JupyterLab 터미널에서)

포드 Start 후 **Connect → JupyterLab** 으로 들어가 터미널에서 (본인 공개키로):

```bash
mkdir -p ~/.ssh && echo "여기에_내_공개키" > ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
```

> 💡 3단계의 "계정 Settings에 SSH 키 등록"을 해뒀으면 이 과정은 **건너뛰어도 돼** (자동 주입됨).

### 2. 복구 + 실행 스크립트 한 줄

처음 세팅할 때 아래 스크립트를 `/workspace/start_comfy.sh` 로 저장해두면 (`/workspace`라 안 날아감),
다음부턴 SSH나 JupyterLab 터미널에서 이 한 줄이면 끝나:

```bash
bash /workspace/start_comfy.sh
```

→ 의존성 설치 + torch 버전 정렬(2.11.0+cu130) + ComfyUI 실행까지 자동. 5~10분 후 `✅ ComfyUI 준비됨!` 뜨면 9단계로.

<details>
<summary>start_comfy.sh 내용 (처음 한 번 /workspace에 저장)</summary>

```bash
#!/bin/bash
set -e

echo '[1/6] 시스템 패키지 (ffmpeg, tmux)...'
command -v ffmpeg >/dev/null 2>&1 && command -v tmux >/dev/null 2>&1 || {
  apt-get update -qq && apt-get install -y -qq ffmpeg tmux
}

echo '[2/6] 코어 의존성...'
cd /workspace/ComfyUI && pip3 install -r requirements.txt --break-system-packages -q
pip3 install sqlalchemy alembic tqdm blake3 --break-system-packages -q

echo '[3/6] 커스텀 노드 의존성...'
cd /workspace/ComfyUI/custom_nodes
for d in */; do
  [ -f "${d}requirements.txt" ] && pip3 install -r "${d}requirements.txt" --break-system-packages -q || true
done
pip3 install opencv-python accelerate soundfile pywavelets scikit-image \
  gguf matplotlib segment-anything piexif ftfy gitpython imageio-ffmpeg --break-system-packages -q

echo '[4/6] diffusers ↔ huggingface_hub 호환 (WanVideoWrapper cached_download 에러 방지)...'
pip3 install -U diffusers --break-system-packages -q

echo '[5/6] torch/torchaudio 정합성 확인...'
python3 -c 'import torch,torchaudio;print("  torch",torch.__version__,"| audio",torchaudio.__version__,"| cuda",torch.cuda.is_available())' || \
  echo '  ⚠️ torchaudio import 실패 → torch에 맞춰 수동 정렬 필요 (6단계 "버전 맞추기" 참고)'

echo '[6/6] ComfyUI 실행 (tmux: comfy)...'
tmux kill-session -t comfy 2>/dev/null || true
tmux new-session -d -s comfy
tmux send-keys -t comfy 'cd /workspace/ComfyUI && python3 main.py --listen 0.0.0.0 --port 8188 2>&1 | tee /workspace/comfyui.log' Enter
echo '서버 뜰 때까지 대기...'
until curl -s http://localhost:8188/system_stats > /dev/null 2>&1; do sleep 3; done
echo '✅ ComfyUI 준비됨! 브라우저에서 [포드ID]-8188.proxy.runpod.net 접속'
```

> 📌 이 스크립트는 **torch를 강제로 재설치하지 않아** (팟마다 기본 torch 버전이 달라서 — 예: cu124 / cu130).
> torchaudio 가 어긋나 import 에러가 나면 [5/6] 단계에서 경고만 띄우니, 그때 6단계 "버전 맞추기"를 수동으로 한 번 해주면 돼.

저장: JupyterLab 터미널에서 위 내용을 `nano /workspace/start_comfy.sh` 로 붙여넣고 저장 → `chmod +x /workspace/start_comfy.sh`
</details>

> 모델 파일이 `/workspace/`에 저장되어 있으면 다시 다운로드 안 해도 돼.
> 단, RunPod 포드를 **완전히 삭제(Terminate)** 하면 `/workspace/`도 날아가니까 주의!

---

## ❓ 자주 생기는 문제

| 문제                                                             | 해결 방법                                                                                           |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `tmux: command not found`                                      | `apt-get update && apt-get install -y tmux` 설치 (8단계 참고)                                         |
| ComfyUI 코어가 안 뜸 + `No module named 'sqlalchemy'`/`'tqdm'`     | 코어 의존성 누락. `pip3 install -r requirements.txt sqlalchemy alembic tqdm blake3 --break-system-packages` |
| ComfyUI가 아예 안 뜸 + `libtorchaudio.so: undefined symbol`         | torch ↔ torchaudio 버전 불일치. 6단계의 "버전 맞추기" 참고                                                     |
| `ffmpeg is required for video outputs` (VHS_VideoCombine)      | ffmpeg 미설치. `apt-get install -y ffmpeg` 후 ComfyUI 재시작 (6단계 ffmpeg 참고)                          |
| WanVideoWrapper 누락 + `cannot import name 'cached_download'`    | diffusers ↔ huggingface_hub 충돌. `pip3 install -U diffusers --break-system-packages` (6단계 참고)    |
| 노드 여러 개가 "누락"으로 뜸                                              | 폴더는 있는데 의존성 미설치일 때 많음. 6단계 의존성(cv2·accelerate·soundfile)·버전정렬 다시 실행 후 ComfyUI 재시작               |
| 어떤 노드가 왜 실패했는지 알고 싶다                                           | `grep -iE "import failed\|no module named\|undefined symbol" /workspace/comfyui.log \| sort -u` |
| `send-keys` 했는데 실행이 안 됨                                        | 명령 줄 끝에 `Enter` 빠졌는지 확인 (8단계 ⚠️)                                                                |
| `Permission denied (publickey)`                                | Stop/Start로 SSH 키가 날아감. JupyterLab에서 키 재등록 (재접속 섹션 1번). 키 문자열이 본인 것과 일치하는지도 확인                  |
| Stop/Start 후 노드 누락·torch 에러 다발                                 | pip 패키지가 초기화된 것. `bash /workspace/start_comfy.sh` 한 줄로 복구 (재접속 섹션 2번)                           |
| 재접속 시 SSH IP·포트가 어제와 다름                                        | Start하면 바뀜. Connect 탭에서 새 주소 확인                                                                 |
| ComfyUI 접속이 안 돼                                                | `tmux attach -t comfy` 로 로그 확인 후 재시작                                                            |
| `ComfyUI-DepthAnythingV3` 만 누락 (`No module named 'comfy_env'`) | AI-RENDERER 워크플로우엔 안 쓰이니 무시해도 됨                                                                 |
| "누락된 노드" 오류                                                    | UI 전용 노드라면 무시하고 실행해도 돼                                                                          |
| 모델 파일 없다고 함                                                    | 7단계 다운로드 다시 확인                                                                                  |
| SSH 접속 안 됨                                                     | RunPod 포드가 켜져 있는지 확인                                                                            |
| `/workspace/comfyui.log: Permission denied`                    | `chmod 777 /workspace/comfyui.log` 실행                                                           |

---

## 📁 이 워크플로우에 필요한 파일 위치 정리

```
/workspace/ComfyUI/
├── models/
│   ├── diffusion_models/wan/
│   │   └── wan-14B_vace_skyreels_v3_R2V_e4m3fn_v1.safetensors  ← 메인 모델
│   ├── text_encoders/
│   │   └── umt5_xxl_fp8_e4m3fn_scaled.safetensors              ← 텍스트 인코더
│   ├── vae/
│   │   └── wan_2.1_vae.safetensors                             ← VAE
│   └── loras/
│       └── Wan2.1_T2V_14B_FusionX_LoRA.safetensors             ← LoRA
└── user/default/workflows/
    └── 260225_MICKMUMPITZ_AI-RENDERER_SMPL_2-0_Runpod.json     ← 워크플로우
```

---

*작성일: 2026-05-16 | 업데이트: 2026-06-11 (새 팟 마이그레이션 — 코어/노드 의존성·diffusers·ffmpeg 누락 반영) | ComfyUI 0.21.1~0.24.0*
