# 🎨 UMMEvalKit

> A unified, efficient, and extensible evaluation toolkit for unified multimodal models

[⚡ Quick Start](#-quick-start) | [中文文档](./README_zh.md) | [English](./README.md) 

---

## 📖 Overview

In the evaluation of unified multimodal models (UMMs), **efficiently evaluating multiple models across multiple benchmarks simultaneously** remains a persistent challenge. Conventional workflows often require writing separate scripts for each *(model, benchmark)* pair, resulting in fragmented pipelines and high maintenance overhead.

To address this issue, we developed **UMMEvalKit**—a general-purpose evaluation toolkit for unified multimodal models on image generation and editing benchmarks. It provides a unified evaluation entry point and configuration interface, supports parallel evaluation across multiple models and benchmarks, and exposes simple yet extensible APIs to facilitate the integration of new models and benchmarks. It can also be used to evaluate standalone image generation and image editing models.

---

## 📑 Contents

- [✨ Key Features](#-key-features)
- [⚡ Quickstart](#-quick-start)
- [🚀 Usage Guide](#-usage-guide)
  - [Environment Setup](#environment-setup)
  - [Launch Command](#launch-command)
    - [1. Parameter Configuration Reference](#1-parameter-configuration-reference)
    - [2. Configuration Priority](#2-configuration-priority)
    - [3. How to Run](#3-how-to-run)
- [🛠️ Development Guide](#️-development-guide)
  - [Integrating a New Model](#integrating-a-new-model)
  - [Integrating a New Benchmark](#integrating-a-new-benchmark)
- [📂 Project Structure](#-project-structure)
- [🔒 Scenarios Where GPU Nodes Cannot Access the Internet](#-scenarios-where-gpu-nodes-cannot-access-the-internet)
- [⭐ Stars](#-stars)
- [👥 Contributors](#-contributors)

---

## ✨ Key Features

### 🎯 1. Unified Evaluation for Multiple Models × Multiple Benchmarks
- In contrast to traditional scripts that typically support only a "single model × single benchmark" setup, UMMEvalKit enables evaluating **multiple models on multiple benchmarks** in a single run.
- Without modifying the codebase, users can flexibly switch and combine models/benchmarks via configuration, substantially reducing script maintenance costs and improving overall evaluation efficiency.

### ⚡ 2. Parallel Evaluation and Resource Utilization
- Supports running multiple evaluation jobs in **parallel** to better utilize compute resources (GPU/CPU), significantly shortening end-to-end evaluation time.
- The degree of parallelism can be configured according to available hardware to balance throughput against per-job GPU memory usage.

### 🧩 3. Modular and Registry-Based Extensibility
- Both models and benchmarks are managed in a **modular, registry-based** manner, enabling straightforward extension of the existing framework.
- To integrate a new model, one only needs to implement a standardized inference interface and register it at the designated location to participate in the unified inference pipeline.
- To integrate a new benchmark, one only needs to implement the corresponding dataset class and evaluation script and register it accordingly—no changes to the core scheduling logic are required—thereby plugging into the full "inference + evaluation" workflow.

### 📊 4. Unified Output Structure and Logging
- Evaluation outputs and logs for different benchmarks are organized under a **unified directory structure**.
- Supports fast lookup of inference outputs and evaluation metrics by "model × benchmark", facilitating comparison, visualization, and debugging.

### 🎭 5. Integrated Generation and Editing Benchmarks
- The toolkit currently integrates **11 generation benchmarks** and **1 editing benchmark**, ready to use after cloning the repository.
- Editing benchmarks commonly include input images and thus can be large in size. To control repository size, only a smaller benchmark, **imgedit**, is retained as a reference implementation. Users may deploy other editing benchmarks based on this example.
- We will continue to add new benchmarks, and community contributions (PRs) are welcome 🎉

---

## ⚡ Quick Start

A minimal runnable example to evaluate `GPT-Image-1.5`'s performance on `Imgedit` via an **OpenAI-compatible API**

### Prerequisites
Before running, ensure you have completed the following configurations in `config.sh`:
1. **API Configuration**: Fill in your `API_KEY` (and `BASE_URL` if you are using a custom API gateway).
2. **Path Configuration**: Set `CONDA_BASE` to your conda installation path.
3. **Environment Mapping**:
   - In `INFER_ENV_MAP`, assign an inference environment for `gpt-image-1.5` (ensure the `openai` library and `Pillow` library are installed).
   - In `EVAL_ENV_MAP`, assign an evaluation environment for `imgedit` (requires installation via `pip install -r requirements/benchmark/edit/imgedit.txt`).

### Run Command
```bash
bash eval.sh --model_names "gpt-image-1.5" --ds_names "imgedit" --use_api true --num_workers 4 --infer true --eval true
```

---

## 🚀 Usage Guide

### Environment Setup

Depending on the **models and benchmarks you plan to evaluate**, locate the corresponding dependency files under `requirements/` and create the appropriate environments. For example, if the model is `Bagel` and the benchmark is `GenEval`, you may run:

```bash
pip install -r requirements/model/bagel.txt
pip install -r requirements/benchmark/t2i/geneval.txt
```

Note: Although the above requirements.txt files have been aligned as closely as possible with the dependency files from the original model or benchmark repositories, the original files themselves may not be perfect. If you encounter installation or runtime errors, please adjust dependencies or versions flexibly according to your specific situation.

---

### Launch Command

> ⚠️ **Before running the main evaluation script `eval.sh`, it is recommended to configure parameters in `config.sh`.**

In addition, some parameters can be temporarily overridden via command-line arguments; see the details below.

---

#### 1. Parameter Configuration Reference

The table below lists the key parameters in `config.sh`, whether they can be overridden by the command line, and their meanings:

| Parameter | Command-line Override | Description | Example |
|----------|------------------------|-------------|---------|
| `DS_NAMES` | ✅ Yes | List of benchmark names to evaluate. Multiple benchmarks are space-separated. If passed via `--ds_names`, the command-line value overrides `config.sh`. | `DS_NAMES=('genai' 'wiseedit')` |
| `MODEL_NAMES` | ✅ Yes | List of model names to evaluate. Multiple models are space-separated. If passed via `--model_names`, the command-line value overrides `config.sh`. | `MODEL_NAMES=('bagel' 'qwen-image')` |
| `CUSTOM_MODEL_KWARGSES` | ✅ Yes | Pass custom parameters for each model, semicolon-separated. | `--custom_model_kwargses "p1=v1;p2=v2"` |
| `USE_API` | ✅ Yes | Whether to use API for inference. | `--use_api true` |
| `NUM_WORKERS` | ✅ Yes | Number of workers for API inference. Only works when `USE_API=true` | `--num_workers 4` |
| `PARALLEL` | ✅ Yes | Whether to enable **parallel evaluation** across model × benchmark combinations. `true` can increase throughput but requires sufficient GPU memory. Currently configurable only in `config.sh`. | `--parallel true` |
| `ENABLE_INFER` | ✅ Yes | Whether to run the inference stage (image generation). If inference outputs already exist, set to `false` to run evaluation only. Can be overridden by `--infer`. | `ENABLE_INFER=true` |
| `ENABLE_EVAL` | ✅ Yes | Whether to run the evaluation stage (metric computation). Can be overridden by `--eval`. | `ENABLE_EVAL=true` |
| `API_KEY` | ❌ No | API key used by benchmarks that rely on external APIs. Currently configurable only in `config.sh`. | `API_KEY="your_api_key_here"` |
| `BASE_URL` | ❌ No | Base URL for API calls required by evaluation. Configurable only in `config.sh`. | `BASE_URL="https://api.openai.com/v1"` |
| `CONDA_BASE` | ❌ No | Conda installation path, used to locate the `python` executable in each evaluation environment. | `CONDA_BASE="/path/to/miniconda3"` |
| `CUDA_BASE` | ❌ No | CUDA installation root, used by some scripts to manually select a CUDA version. | `CUDA_BASE="/path/to/cuda"` |
| `TORCH_HOME` | ❌ No | PyTorch model cache directory. | `TORCH_HOME="/path/to/.cache/torch"` |
| `HF_HOME` | ❌ No | HuggingFace root cache directory; relevant models will be cached here. | `HF_HOME="/path/to/UMMEvalKit/.cache/huggingface"` |
| `TRANSFORMERS_CACHE` | ❌ No | Transformers cache directory, typically `${HF_HOME}/hub`. | `TRANSFORMERS_CACHE="$HF_HOME/hub"` |
| `HF_HUB_OFFLINE` | ❌ No | Whether to enable HuggingFace offline mode. `1` means **fully offline** (no downloads); `0` allows downloads. | `HF_HUB_OFFLINE=1` |
| `TRANSFORMERS_OFFLINE` | ❌ No | Whether to enable Transformers offline mode; semantics match `HF_HUB_OFFLINE`. | `TRANSFORMERS_OFFLINE=1` |
| `INFER_ENV_MAP` | ❌ No | Mapping from model name to conda environment name for the **inference stage**. Keys are model names and values are inference environment names. Currently configurable only in `config.sh`. | `INFER_ENV_MAP=(['bagel']='bagel-env' ['qwen-image']='qwen-image-env')` |
| `EVAL_ENV_MAP` | ❌ No | Mapping from benchmark name to conda environment name(s) for the **evaluation stage**. If multiple environments are required, separate them with spaces. | `EVAL_ENV_MAP=(['genai']='yph-genai' ['wiseedit']='yph-risebench')` |
| `EVAL_GPU_MAP` | ❌ No | Number of GPUs required by each benchmark during the **evaluation stage**. Keys are benchmark names and values are GPU counts; `0` indicates no GPU usage (e.g., API-only evaluation). Used primarily to inform schedulers (e.g., `srun`) about required GPU resources. | `EVAL_GPU_MAP=(['genai']=1 ['gedit']=0 ['cvtg']=1)` |
| `GENERATE_BLANK_IMAGE_ON_ERROR` | ❌ No | If an exception occurs during inference, whether to automatically generate a **blank placeholder image** to prevent a single failed sample from interrupting the entire evaluation run. Configurable only in `config.sh`. | `GENERATE_BLANK_IMAGE_ON_ERROR=false` |

> 💡 **Note:**  
> - “Command-line override” means that when the corresponding argument is provided (e.g., `--ds_names genexam` / `--model_names internvl-u` ), the **command-line value takes precedence** over `config.sh`.  
> - Other parameters must be modified in `config.sh`.

---

#### 2. Configuration Priority

The overall priority order is:

1. `config.sh`:  
   - Provides **default settings**, suitable for relatively stable, routine evaluation scenarios.  
   - It is recommended to place long-term configurations such as models/benchmarks/environment mappings here.

2. Command-line arguments:  
   - Used to temporarily change evaluation scope (e.g., evaluating only a subset of models or benchmarks).  
   - Effective only for parameters that support overriding: `ds_names`, `model_names`, `infer`, and `eval`.

If the same parameter is specified in both places, **the command-line argument takes precedence** and overrides the corresponding entry in `config.sh`.

Example:

- In `config.sh`:  
  ```bash
  DS_NAMES=('genai' 'wiseedit')
  ```
- In the command line:  
  ```bash
  --ds_names "wiseedit;gedit"
  ```

Then the benchmarks evaluated in this run will be `wiseedit` and `gedit`.

---

#### 3. How to Run

**📌 Option 1: Use the configuration in `config.sh` directly**

After completing the basic setup in `config.sh`, run the main evaluation script:

```bash
bash eval.sh
```

In this mode:

- The **model list** is determined by `MODEL_NAMES`.  
- The **benchmark list** is determined by `DS_NAMES`.  
- Whether to run inference/evaluation is controlled by `ENABLE_INFER` and `ENABLE_EVAL`, respectively.  
- The specific environments and GPU requirements used for inference/evaluation are determined by `INFER_ENV_MAP`, `EVAL_ENV_MAP`, `EVAL_GPU_MAP`, etc.

This is suitable for routine evaluations with stable configurations, reducing the need to repeatedly specify command-line parameters.

---

**📌 Option 2: Temporarily override part of the configuration via command-line arguments**

If you prefer not to modify `config.sh` and only want to change the evaluation scope or switches for a single run, you can pass command-line arguments:

```bash
bash eval.sh \
  --ds_names "wiseedit;gedit" \
  --model_names "bagel;qwen-image" \
  --custom_model_kwargses ";" \
  --infer true \
  --eval true \
```

This command means:

- Evaluate only the two models `bagel` and `qwen-image`.  
- Evaluate the benchmarks `wiseedit` and `gedit`.  
- Run both inference (`infer=true`) and evaluation (`eval=true`) regardless of the defaults in `config.sh`.

---

## Development Guide

This section describes how to integrate a new model and a new benchmark into UMMEvalKit.

### Integrating a New Model

#### Step 1. Prepare Model Weights

It is recommended to place model weights under:

```text
infer/custom_models/checkpoints/
```

This makes model invocation convenient. Alternatively, you may store weights elsewhere, but you must specify the correct path when registering the model loading logic.

---

#### Step 2. Implement the Model Invocation Interface

Under `infer/custom_models/model_utils`, create a new interface file for your model and implement a model class. The class must satisfy the following requirements:

- **Required attribute:**  
  - `self.model_name`: a string used to identify the model (must match, or be mappable to, the name in `MODEL_NAMES`).

- **Required method:**  
  - `self.generate()`: the core inference function.

The `generate` interface depends on the task type:

- **Text-to-Image (t2i)**

  ```python
  def generate(self, prompt, seed=42):
      """
      prompt: text prompt for text-to-image generation
      seed:   random seed for inference
      """
      ...
  ```

- **Image Editing (edit)**

  ```python
  def generate(self, input_list, seed=42):
      """
      input_list: inputs required for image editing, typically a list of (str, PIL.Image),
                  including editing instructions and the source image, etc.
      seed:       random seed for inference
      """
      ...
  ```

---

#### Step 3. Register the Model Loading Logic

After defining the model class, register it under the `MODEL_SETTINGS` variable in:

```text
UMMEvalKit/infer/custom_models/load_model.py
```

so that the main evaluation pipeline can automatically load the model by name.

Common configuration fields include:

| Field | Description |
|------|-------------|
| `model_path` | Path to model weights (typically under `infer/custom_models/checkpoints/`). |
| `module` | Module path of the model interface (Python import path). |
| `class` | Model class name (as defined in the corresponding `.py` file under `model_utils`). |
| `model_kwargs` | Additional initialization parameters (`dict`). If user-provided custom parameters with the same keys are supplied in `config.sh` or via the command line, they override these defaults. |

After completing the above steps, you can include the model name in `MODEL_NAMES` in `config.sh` or pass it via `--model_names` in the command line to participate in evaluation.

---

## Integrating a New Benchmark

#### Step 1. Define the Dataset Class

Create the dataset class under the following directory, depending on the task type (t2i/edit):

```text
infer/custom_datasets/dataset_cls/edit/
infer/custom_datasets/dataset_cls/t2i/
```

The dataset class must include:

- **Required attributes:**  
  - `self.dataset_name`: identifies the benchmark name.  
  - `self.dataset_task`: identifies the task type (e.g., `"t2i"` or `"edit"`).  
  - `self.data`: a list of evaluation samples.

Here, `self.data` is a `list` of `dict`, where each element represents one evaluation instruction with the following required structure:

- **For t2i tasks:**

  ```python
  {
      "prompt": ... ,          # text prompt
      "seed": ... ,            # random seed; can vary to generate multiple images per prompt
      "image_save_path": ... , # path to save the output image
  }
  ```

- **For edit tasks:**

  ```python
  {
      "input_list": ... ,      # list combining the source image and edit instruction, typically list(str, PIL.Image)
      "seed": ... ,            # random seed; can vary to generate multiple edited outputs
      "image_save_path": ... , # path to save the output image
  }
  ```

After implementing the dataset class, register the benchmark in:

```text
UMMEvalKit/infer/custom_datasets/load_dataset.py
```

so that the main pipeline can automatically load the dataset by benchmark name.

---

#### Step 2. Implement the Evaluation Script

Following the official evaluation protocol/command provided by the benchmark, implement an evaluation script and place it under:

```text
UMMEvalKit/eval/
```

The script should accept at least the following input parameters:

- `MODEL_NAME`: name of the model to evaluate.  
- `EVAL_ENV`: conda environment name required for running evaluation (typically obtained from `EVAL_ENV_MAP`).

#### Step 3. Configure Evaluation Parameters

In `config.sh`:

- Set the corresponding conda environment name under `EVAL_ENV_MAP`.  
- Set the required number of GPUs for evaluation under `EVAL_GPU_MAP`.

> Important:  
> - During **model training**, the working directory is typically the repository root (e.g., `UMMEvalKit`).  
> - During **evaluation**, the working directory will be switched to the benchmark repository directory.  
>   When writing evaluation scripts, pay careful attention to how paths are resolved.

---

## Project Structure

The main directory structure and its functionality are as follows:

```text
📦 UMMEvalKit
├── 📁 benchmarks/          # Evaluation repositories for each benchmark
├── 📁 eval/                # Evaluation script entries (organized by benchmark)
├── 📁 eval_models/         # Model weights for evaluation (e.g., CLIP)
├── 📁 infer/               # Inference interface layer
│   ├── 📁 custom_datasets/ # Benchmark dataset interfaces
│   └── 📁 custom_models/
│       ├── 📁 checkpoints/ # Model weight files
│       └── 📁 model_utils/ # Model invocation interfaces
├── 📁 log/                 # Inference and evaluation logs
├── 📁 output/              # Generated images and evaluation results (organized by model and benchmark)
├── 📁 requirements/        # Dependencies for model inference and benchmark evaluation
└── 📁 utils/               # Utility scripts
    ├── check_empty_image.py      # detect blank output images (useful for diagnosing failures)
    ├── download_model_hf_home.sh # manually download HuggingFace models into HF_HOME
    └── use_cuda.sh               # switch CUDA versions
```

---

## 🔒 Scenarios Where GPU Nodes Cannot Access the Internet

Some benchmarks download models from HuggingFace before deploying them locally for GPU inference. If **GPU nodes cannot directly access the public internet**, required weights must be downloaded in an internet-accessible environment and then transferred to the evaluation environment.

In this project, you can manually download models into `${HF_HOME}/hub` (where `HF_HOME` is configured in `eval.sh`) via:

- Using the script `utils/download_model_hf_home.sh`:  
  - Run the script in an internet-accessible environment to download the required models into the specified `HF_HOME`.  
  - Synchronize/copy the cache directory to the offline GPU environment.

- For non-HuggingFace models:  
  - Follow the official download procedure provided by the model authors; download in an internet-accessible environment and place the weights under `eval_models` or the appropriate subdirectory of `$HOME/.cache`, then synchronize to the evaluation environment.

After downloading all required artifacts, set `HOME`, `HF_HOME`, and `TRANSFORMERS_CACHE` in `config.sh`, and enable offline mode by setting `HF_HUB_OFFLINE=1` and `TRANSFORMERS_OFFLINE=1` to complete evaluation in offline settings.

---

## ⭐ Stars

If you find this toolkit helpful, please consider starring the repository! ⭐

[![Star History Chart](https://api.star-history.com/svg?repos=PenghaoYin/UMMEvalKit&type=Date)](https://star-history.com/#PenghaoYin/UMMEvalKit&Date)

---

## 👥 Contributors

<table>
  <tr>
    <td align="center" width="180">
      <a href="https://github.com/PenghaoYin">
        <img src="https://github.com/PenghaoYin.png?size=120" width="96" height="96" alt="PenghaoYin" />
        <br />
        <sub><b>Penghao Yin</b></sub>
      </a>
      <br />
      <sub>🚀 Lead Developer</sub>
    </td>
    <td align="center" width="180">
      <a href="https://github.com/nini0919">
        <img src="https://github.com/nini0919.png?size=120" width="96" height="96" alt="nini0919" />
        <br />
        <sub><b>Danni Yang</b></sub>
      </a>
      <br />
      <sub>💡 Contributor</sub>
    </td>
    <td align="center" width="180">
      <a href="https://github.com/Ganlin-Yang">
        <img src="https://github.com/Ganlin-Yang.png?size=120" width="96" height="96" alt="Ganlin-Yang" />
        <br />
        <sub><b>Ganlin Yang</b></sub>
      </a>
      <br />
      <sub>💡 Contributor</sub>
    </td>
    <td align="center" width="180">
      <a href="https://github.com/ChangyaoTian">
        <img src="https://github.com/ChangyaoTian.png?size=120" width="96" height="96" alt="ChangyaoTian" />
        <br />
        <sub><b>Changyao Tian</b></sub>
      </a>
      <br />
      <sub>🎯 Advisor</sub>
    </td>
  </tr>
</table>

---

<div align="center">

**Made with ❤️ by the UMMEvalKit Team**

</div>