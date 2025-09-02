# Orpheus-CPP — Setup on Windows 11 Pro & Ubuntu

This guide installs **Orpheus-CPP**, the **FastRTC demo**, and **llama-cpp-python** using a Python **3.11** virtual environment (recommended for best wheel availability on both Windows and Linux).

> If you’re using **Python 3.12**, pip may try to build `llama-cpp-python` from source. Prefer **Python 3.11** to use prebuilt wheels where possible.

---

## Contents

- [Windows 11 Pro](#windows-11-pro)
  - [1) Project folder & Python 3.11 venv](#1-project-folder--python-311-venv)
  - [2) Upgrade pip tooling](#2-upgrade-pip-tooling)
  - [3) Install Orpheus + dependencies](#3-install-orpheus--dependencies)
  - [4) Run the FastRTC demo](#4-run-the-fastrtc-demo)
  - [5) Troubleshooting (Windows)](#5-troubleshooting-windows)
  - [6) VS Code tips (Windows)](#6-vs-code-tips-windows)
- [Ubuntu (22.04 / 24.04)](#ubuntu-2204--2404)
  - [1) System prep & venv](#1-system-prep--venv)
  - [2) Install Orpheus + dependencies](#2-install-orpheus--dependencies-1)
  - [3) Run the demo](#3-run-the-demo)
  - [4) Troubleshooting (Ubuntu)](#4-troubleshooting-ubuntu)
- [Programmatic TTS example](#programmatic-tts-example)
- [Optional: Requirements file](#optional-requirements-file)

---

## Windows 11 Pro

### 1) Project folder & Python 3.11 venv

Open **PowerShell** and clone the repo and go to project (adjust the path to your folder if needed):

```powershell
git clone git@github.com:letebrhan/orpheus-cpp.git
cd orpheus-cpp
```

Create & activate a **Python 3.11** venv:

```powershell
py -3.11 --version
# If missing: winget install Python.Python.3.11

py -3.11 -m venv venv311

# If PowerShell blocks activation, allow scripts for this session:
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

.\venv311\Scripts\Activate.ps1
```

### 2) Upgrade pip tooling

```powershell
python -m pip install --upgrade pip wheel setuptools
```

### 3) Install Orpheus + dependencies

```powershell
pip install orpheus-cpp
pip install "fastrtc>=0.0.17"

# Prefer prebuilt CPU wheels for llama-cpp-python (Windows/Linux):
pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu
```

### 4) Run the FastRTC demo

```powershell
python -m orpheus_cpp
```

Open: <http://localhost:7860> (allow Windows Firewall if prompted).

### 5) Troubleshooting (Windows)

**A) GGUF error (e.g., “wrong number of tensors; expected … got …”)**  
This usually means the downloaded GGUF doesn’t match the llama.cpp runtime or the file was corrupted. Clear the specific cached model so it re-downloads:

```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.cache\huggingface\hub\models--isaiahbjork--orpheus-3b-0.1-ft-Q4_K_M-GGUF"
python -m orpheus_cpp
```

If needed, you can clear the whole HF cache (heavier):
```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.cache\huggingface\hub"
```

**B) Pip tries to compile `llama-cpp-python` (CMake/NMake/MSVC errors)**  
Fastest fix: use Python **3.11** (above). If it still compiles:

- Install **CMake** (if needed):
  ```powershell
  winget install Kitware.CMake
  ```
- Install **Visual Studio Build Tools** (no full IDE required). In the installer, select the workload:
  - **Desktop development with C++**
  - Ensure these components are selected: **MSVC v143**, **Windows 11 SDK**, **C++ CMake tools for Windows**.
- Open **x64 Native Tools Command Prompt for VS 2022** and run:
  ```cmd
  pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu
  ```

Alternatively, try a nearby pinned version (sometimes wheels exist for a slightly earlier release):
```powershell
pip install "llama-cpp-python==0.3.2" --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu
```

**C) Activation blocked in VS Code terminal**  
Use a per-session bypass before activating:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\venv311\Scripts\Activate.ps1
```

### 6) VS Code tips (Windows)

- **Select interpreter**: `Ctrl+Shift+P` → *Python: Select Interpreter* → choose `.\venv311\Scripts\python.exe`.
- **Alternate terminal**: If preferred, switch VS Code terminal to **Command Prompt** and activate with:
  ```cmd
  venv311\Scripts\activate.bat
  ```

---

## Ubuntu (22.04 / 24.04)

### 1) System prep & venv

```bash
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3-pip

git clone git@github.com:letebrhan/orpheus-cpp.git
cd orpheus-cpp
python3.11 -m venv venv311
source venv311/bin/activate
python -m pip install --upgrade pip wheel setuptools
```

### 2) Install Orpheus + dependencies

```bash
pip install orpheus-cpp
pip install "fastrtc>=0.0.17"

# Prefer prebuilt CPU wheels for llama-cpp-python (Windows/Linux):
pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu
```

### 3) Run the demo

```bash
python -m orpheus_cpp
```

Open: <http://localhost:7860>

### 4) Troubleshooting (Ubuntu)

**A) GGUF error (tensor mismatch)**

```bash
rm -rf ~/.cache/huggingface/hub/models--isaiahbjork--orpheus-3b-0.1-ft-Q4_K_M-GGUF
python -m orpheus_cpp
```

(Or clear the whole cache with `rm -rf ~/.cache/huggingface/hub` if necessary.)

**B) Pip tries to compile `llama-cpp-python`**

- Ensure Python **3.11** is active (as above).
- If it still compiles, install build tools and retry:
  ```bash
  sudo apt install -y build-essential cmake
  pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu
  ```
- Or avoid compilers completely with **Conda**:
  ```bash
  conda create -n orpheus311 python=3.11
  conda activate orpheus311
  conda install -c conda-forge llama-cpp-python
  pip install orpheus-cpp "fastrtc>=0.0.17"
  python -m orpheus_cpp
  ```

---

## Programmatic TTS example

```python
from orpheus_cpp import OrpheusCpp
from scipy.io.wavfile import write

orpheus = OrpheusCpp()
text = "Hello from Orpheus!"
sample_rate, samples = orpheus.tts(text, options={"voice_id": "tara"})
write("output.wav", sample_rate, samples.squeeze())
```

---

## Optional: Requirements file

If you want to pin a working combo, create a `requirements.txt` like:

```
# Core
orpheus-cpp==0.0.3
fastrtc>=0.0.17

# Prefer a known-good llama.cpp runtime (adjust if you need newer)
llama-cpp-python==0.3.2

# Typical deps pulled by orpheus-cpp
numpy>=2.0
transformers>=4.56.0
onnxruntime>=1.22.1
huggingface-hub>=0.34.4
tqdm>=4.67.0
```

Install with:

```bash
pip install -r requirements.txt
```

---

**Tip:** If a future update causes GGUF load errors, first clear the model’s cache directory and re-run. If needed, pin `llama-cpp-python` to a nearby version that works for your hardware/OS combo.
