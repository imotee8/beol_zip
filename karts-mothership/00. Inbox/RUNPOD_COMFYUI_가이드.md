---
related:
  - "[[ComfyUI]]"
  - "[[🐝 Beol.zip]]"
---
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
mkdir -p ~/.ssh && echo "여기에_위에서_복사한_공개키_붙여넣기" >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
```

mkdir -p ~/.ssh && echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIImTonazMfUfybwPnIhF52vagf4xhxCJKH6DGy7Ogrsl cm.kafa.01@gmail.com" >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
---

## 4단계. SSH 접속 정보 확인하고 접속하기

RunPod **Connect** 탭에 두 가지 접속 방법이 나와:

```
SSH:              ssh j515...@ssh.runpod.io -i ~/.ssh/id_ed25519
SSH over TCP:     ssh root@213.xxx.xxx.xx -p 36190 -i ~/.ssh/id_ed25519
```

> **TCP 방식 추천** (파일 전송도 가능해)

터미널에서 접속 테스트:
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

tmux를 써야 SSH 끊겨도 ComfyUI가 계속 살아있어:

```bash
tmux new-session -d -s comfy
tmux send-keys -t comfy 'cd /workspace/ComfyUI && python3 main.py --listen 0.0.0.0 --port 8188 2>&1 | tee /workspace/comfyui.log' Enter
```

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

## 🔄 다음에 다시 시작할 때 (재접속)

포드를 껐다 켰거나 SSH 재접속 후에는 ComfyUI만 다시 실행하면 돼:

```bash
# tmux 세션이 살아있는지 확인
tmux ls

# 살아있으면 그냥 접속
tmux attach -t comfy

# 죽어있으면 다시 시작
tmux new-session -d -s comfy
tmux send-keys -t comfy 'cd /workspace/ComfyUI && python3 main.py --listen 0.0.0.0 --port 8188 2>&1 | tee /workspace/comfyui.log' Enter
```

> 모델 파일이 `/workspace/`에 저장되어 있으면 다시 다운로드 안 해도 돼.  
> 단, RunPod 포드를 **완전히 삭제**하면 `/workspace/`도 날아가니까 주의!

---

## ❓ 자주 생기는 문제

| 문제 | 해결 방법 |
|------|-----------|
| ComfyUI 접속이 안 돼 | `tmux attach -t comfy` 로 로그 확인 후 재시작 |
| "누락된 노드" 오류 | UI 전용 노드라면 무시하고 실행해도 돼 |
| 모델 파일 없다고 함 | 7단계 다운로드 다시 확인 |
| SSH 접속 안 됨 | RunPod 포드가 켜져 있는지 확인 |
| `/workspace/comfyui.log: Permission denied` | `chmod 777 /workspace/comfyui.log` 실행 |

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

*작성일: 2026-05-16 | RunPod 포드 ID: j515lhdqjg16k0 | ComfyUI 0.21.1*
