# 환경설정 방법
# ComfyUI Hunyuan 3D Asset Pipeline Setup

이 프로젝트는 **ComfyUI Windows Portable** 환경에서 텍스트 및 이미지를 3D 모델로 변환하는 **Hunyuan 3D** 워크플로우 구축 가이드입니다.

---

## 목차

1. [사전 요구 사항](#1-사전-요구-사항)
2. [ComfyUI 설치](#2-comfyui-설치)
3. [모델 설치 및 파일 배치](#3-모델-설치-및-파일-배치)
4. [커스텀 노드 환경 설정](#4-커스텀-노드-환경-설정)

---

## 1. 사전 요구 사항

이 워크플로우의 핵심인 `Hunyuan 3D` 노드는 C++ 컴파일이 필요하므로, 다음 소프트웨어가 반드시 설치되어 있어야 합니다.

* **Visual Studio Build Tools 2022:** [다운로드](https://visualstudio.microsoft.com/ko/downloads/)
    * 설치 시 워크로드 탭에서 "C++를 사용한 데스크톱 개발"을 반드시 체크하여 설치하세요.
* **NVIDIA CUDA Toolkit 12.1:** [다운로드](https://developer.nvidia.com/cuda-12-1-0-download-archive)
    * GPU 가속을 위해 필요합니다.

---

## 2. ComfyUI 설치

1.  [ComfyUI GitHub Releases](https://github.com/comfyanonymous/ComfyUI/releases) 페이지에서 최신 **Windows Portable (7z)** 파일을 다운로드합니다.
2.  원하는 위치에 압축을 풉니다. (예: `D:\ComfyUI_windows_portable\`)
3.  이후 모든 작업은 이 폴더를 기준으로 진행됩니다.

---

## 3. 모델 설치 및 파일 배치

아래 모델들을 다운로드하여 `ComfyUI` 내부의 지정된 경로에 배치합니다. 폴더가 없다면 생성해야 합니다.

| 종류 | 모델명 | 저장 위치 (`ComfyUI/`) | 다운로드 링크 |
| :--- | :--- | :--- | :--- |
| **Checkpoint** | `DreamShaperXL_Lightning.safetensors` | `models/checkpoints/` | [HuggingFace](https://huggingface.co/Lykon/dreamshaper-xl-lightning/tree/main) |
| **ControlNet** | `diffusion_pytorch_model.safetensors` | `models/controlnet/` | [HuggingFace](https://huggingface.co/TencentARC/t2i-adapter-openpose-sdxl-1.0/tree/main) |
| **Diffusion** | `hunyuan3d-dit-v2-0-fp16.safetensors` | `models/diffusion_models/` | [HuggingFace](https://huggingface.co/Kijai/Hunyuan3D-2_safetensors/tree/main) |
| **Upscale** | `RealESRGAN_x4plus_anime_6B.pth` | `models/upscale_models/` | [HuggingFace](https://huggingface.co/ac-pill/upscale_models/tree/main) |

---

## 4. 커스텀 노드 환경 설정

`Hunyuan 3D Wrapper` 노드의 핵심인 `custom_rasterizer` 모듈을 포터블 환경에서 빌드하기 위한 과정입니다.

### Step 1: 커스텀 노드 복제
`ComfyUI/custom_nodes/` 폴더에서 터미널을 열고 아래 명령어를 순서대로 실행합니다.
1. git clone https://github.com/kijai/ComfyUI-Hunyuan3DWrapper
2. cd ComfyUI-Hunyuan3DWrapper
3. pip install -r requirements.txt
4. python_embeded\python.exe -m pip install ComfyUI\custom_nodes\ComfyUI-Hunyuan3DWrapper\wheels\custom_rasterizer-0.1.0+torch260.cuda126-cp312-cp312-win_amd64.whl

---

# 프로젝트 문제 해결

이 프로젝트는 `ComfyUI-Hunyuan3DWrapper` 커스텀 노드를 ComfyUI Portable 환경에 설치하고 실행하는 과정에서 발생한 기술적 문제와 해결 과정을 기록합니다.

---

## 1. 핵심 설치 오류: Python 버전 비호환
* **문제:** `custom_rasterizer` 모듈 컴파일 실패
    * **근본 원인:** 사용 중인 ComfyUI 포터블 버전이 **Python 3.13** (`python_embeded`)을 기반으로 하고 있으나, `ComfyUI-Hunyuan3DWrapper` 커스텀 노드는 **Python 3.12** 환경에 맞춰 빌드되어야 했습니다.
    * **증상:** 시스템에 Python 3.13 개발 파일을 강제로 설치하고 경로를 지정해도, 3.12용으로 작성된 C++ 코드가 3.13 환경에서 호환되지 않아 컴파일에 실패합니다.
    * **해결:** ComfyUI의 Python 3.13 포터블 환경을 유지하면서, 누락된 C++ 빌드용 개발 파일(include, libs)을 수동으로 추가하여 문제를 해결했습니다.
        1. Hunyuan3DWrapper의 기본 Python 의존성을 설치합니다.
        - ComfyUI\custom_nodes\ComfyUI-Hunyuan3DWrapper 폴더에서 cmd 실행: ..\..\python_embeded\python.exe -m pip install -r requirements.txt
        2. Python 3.13용 개발 파일(https://github.com/woct0rdho/triton-windows?tab=readme-ov-file#8-special-notes-for-comfyui-with-embeded-python 링크의 "python_3.13.2_include_libs.zip")을 별도로 다운로드합니다.
        3. 다운로드한 include 폴더와 libs 폴더를 ComfyUI\python_embeded\ 경로에 붙여넣습니다.
        4. custom_rasterizer 소스 코드 경로로 이동하여 C++ 코드를 수동으로 빌드합니다. (이 단계에서는 Visual Studio Developer Command Prompt 사용을 권장합니다.)
        - ComfyUI\custom_nodes\ComfyUI-Hunyuan3DWrapper\hy3dgen\texgen\custom_rasterizer 폴더에서 cmd 실행: ..\..\..\..\python_embeded\python.exe setup.py build_ext --inplace
        5. 빌드가 성공하면 ...custom_rasterizer 폴더 내에 생성된 .pyd 파일 (예: custom_rasterizer_kernel.cp313-win_amd64.pyd)을 복사합니다.
        6. 복사한 .pyd 파일을 ComfyUI가 모듈을 인식할 수 있도록 ComfyUI\python_embeded\Lib\site-packages\ 폴더에 붙여넣습니다.

## 2. GPU 및 드라이버 오류
* **문제: CUDA Toolkit 미설치**
    * **증상:** `Hunyuan 3D` 노드 실행 시 CUDA 관련 오류가 발생하며 GPU를 인식하지 못했습니다.
    * **해결:** `Hunyuan 3D` 노드는 GPU 연산을 위해 NVIDIA CUDA Toolkit이 필요합니다. 사용중인 GPU에 맞는 CUDA Toolkit을 설치하여 시스템 환경을 구성했습니다.
