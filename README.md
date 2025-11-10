# 🔧 프로젝트 문제 해결 (Troubleshooting)

이 프로젝트는 `ComfyUI-Hunyuan3DWrapper` 커스텀 노드를 ComfyUI Portable 환경에 설치하고 실행하는 과정에서 발생한 기술적 문제와 해결 과정을 기록합니다.

---

## 1. 핵심 설치 오류: Python 버전 비호환
* **문제:** `custom_rasterizer` 모듈 컴파일 실패
* **근본 원인:** 사용 중인 ComfyUI 포터블 버전이 **Python 3.13** (`python_embeded`)을 기반으로 하고 있으나, `ComfyUI-Hunyuan3DWrapper` 커스텀 노드는 **Python 3.12** 환경에 맞춰 빌드되어야 했습니다.
* **주요 증상:**
    1.  **`Python.h` 없음:** 포터블 버전에는 C++ 빌드에 필요한 Python 개발 헤더 파일이 없습니다.
    2.  **컴파일 실패:** 시스템에 Python 3.13 개발 파일을 강제로 설치하고 경로를 지정해도, 3.12용으로 작성된 C++ 코드가 3.13 환경에서 호환되지 않아 컴파일에 실패합니다.
* **해결:**
* ComfyUI의 Python 3.13 포터블 환경을 유지하면서, 누락된 C++ 빌드용 개발 파일(include, libs)을 수동으로 추가하여 문제를 해결했습니다.
    1. Hunyuan3DWrapper의 기본 Python 의존성을 설치합니다.
    - ComfyUI\custom_nodes\ComfyUI-Hunyuan3DWrapper 폴더에서 cmd 실행: ..\..\python_embeded\python.exe -m pip install -r requirements.txt
    2. Python 3.13용 개발 파일(https://github.com/woct0rdho/triton-windows?tab=readme-ov-file#8-special-notes-for-comfyui-with-embeded-python 링크의 "python_3.13.2_include_libs.zip")을 별도로 다운로드합니다.
    3. 다운로드한 include 폴더와 libs 폴더를 ComfyUI\python_embeded\ 경로에 복사합니다.
    4. custom_rasterizer 소스 코드 경로로 이동하여 C++ 코드를 수동으로 빌드합니다. (이 단계에서는 Visual Studio Developer Command Prompt 사용을 권장합니다.)
    - ComfyUI\custom_nodes\ComfyUI-Hunyuan3DWrapper\hy3dgen\texgen\custom_rasterizer 폴더에서 cmd 실행: ..\..\..\..\python_embeded\python.exe setup.py build_ext --inplace
    5. 빌드가 성공하면 ...custom_rasterizer 폴더 내에 생성된 .pyd 파일 (예: custom_rasterizer_kernel.cp313-win_amd64.pyd)을 복사합니다.
    6. 복사한 .pyd 파일을 ComfyUI가 모듈을 인식할 수 있도록 ComfyUI\python_embeded\Lib\site-packages\ 폴더에 붙여넣습니다.

## 2. GPU 및 드라이버 오류
* **문제: CUDA Toolkit 미설치**
    * **증상:** `Hunyuan 3D` 노드 실행 시 CUDA 관련 오류가 발생하며 GPU를 인식하지 못했습니다.
    * **해결:** `Hunyuan 3D` 노드는 GPU 연산을 위해 NVIDIA CUDA Toolkit이 필요합니다. 사용중읜 GPU에 맞는 CUDA Toolkit을 설치하여 시스템 환경을 구성했습니다.
