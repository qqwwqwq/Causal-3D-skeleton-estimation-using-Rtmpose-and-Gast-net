



# 🚀 Real-Time 3D Pose Estimation Pipeline

A high-performance, low-latency real-time single-person 3D pose estimation and rendering pipeline built on **RTMDet + RTMPose + GAST-Net**.

This project implements an end-to-end lift from 2D video to a 3D skeleton, featuring a deep engineering refactoring to eliminate performance bottlenecks in traditional Python video processing, supporting smooth real-time visualization and video export.
---

## 📥 Download Full Project

Due to GitHub's file size limits (especially for model weights and environment files), the complete project including all necessary assets has been uploaded to Google Drive. 

**Please download the entire project folder from the link below:**
👉 **[Download Project via Google Drive](https://drive.google.com/file/d/1DDL6_HJ7lttEttyCDnJLJ0h1dEzPXSgq/view?usp=sharing)**

---
https://github.com/user-attachments/assets/5059373f-99a5-46fa-867d-7044986dfed1
## ✨ Key Features

* **⚡ Asynchronous Multi-Threaded Architecture (Pipeline)**: Completely decouples the "video reading", "AI inference", and "frame rendering" modules, eliminating I/O blocking and maximizing GPU utilization.
* **⏱️ Strict Causal Convolution for Zero Algorithmic Latency**: Unlike traditional offline 3D models that rely on future frames (introducing unavoidable buffering delays), this pipeline utilizes GAST-Net's strictly causal model (`causal=True`). It predicts the 3D pose relying **only on past and present frames**, eliminating algorithmic latency for delay-sensitive applications like live VTubing and robotics.
* **🏎️ Ultra-Fast Pure OpenCV 3D Rendering**: Strips away the heavily time-consuming native Matplotlib rendering. It directly extracts Matplotlib's underlying 3D projection matrix and uses OpenCV's `cv2.line` for sub-pixel hard rendering, dropping single-frame rendering latency from ~30ms to **<0.5ms**.
* **🎯 Accurate Target Tracking & Stabilization**:
    * Integrates `ByteTracker` for consistent target identity (ID).
    * Integrates a `One-Euro Filter` for temporal filtering to eliminate high-frequency jitter.


* **🦴 Smart Mapping from Halpe-26 to H36M**: Natively supports RTMPose's Halpe format and leverages its highly accurate Head Top keypoint (Index 17) to map directly to Human3.6M format, ensuring much better head positioning than traditional estimations.
* **🛡️ Crash-Proof Video Export**: Built-in adaptive padding logic fills a blank background when a target is lost, preventing FFmpeg `Failed to write frame` errors caused by resolution changes.

## 🎨 Visualization

The 3D skeleton uses a high-contrast anatomical color scheme:
* **Black (#000000)**: Left side limbs and pelvis.
* **Red (#FF0000)**: Right side limbs.
* **Lime Green (#00FF00)**: Trunk centerline (spine, neck, head).



## 🛠️ Installation

This project relies on a specific environment configuration for stability. 

**Applicable Environment**: Windows / Linux (GPU with CUDA 11.7 support)  
**Core Versions**: Python 3.8 + PyTorch 1.13.1 + NumPy 1.23.5

> 💡 **Quick Setup:** For your convenience, we provide both `environment.yml` and `requirements.txt` in the repository. You can quickly set up the base environment using:
> * **Conda:** `conda env create -f environment.yml`
> * **Pip:** `pip install -r requirements.txt`
> 
> *(Note: After using the quick setup, please proceed to Steps 4 and 5 to install OpenMMLab core libraries.)*

**Alternatively, follow the step-by-step manual installation:**

**1. Create and Activate Environment**
```bash
# Python 3.8 is the best version for backward compatibility
conda create -n realtime_pose python=3.8 -y
conda activate realtime_pose
```

**2. Install PyTorch**
(PyTorch 1.13.1 is the final stable 1.x release with native CUDA 11.7 support)
```bash
conda install pytorch==1.13.1 torchvision==0.14.1 torchaudio==0.13.1 pytorch-cuda=11.7 -c pytorch -c nvidia -y
```

**3. Install Dependencies**
(⚠️ Strictly pin NumPy to 1.23.5 to prevent 3D code runtime errors)
```bash
pip install numpy==1.23.5 scipy h5py matplotlib opencv-python tqdm
pip install torchsummary
```

**4. Install OpenMMLab Engine**
```bash
pip install -U openmim
mim install mmengine
```

**5. Install Pose Estimation Core Libraries**
(⚠️ Strictly pin versions to avoid dependency conflicts)
```bash
mim install "mmcv==2.1.0"
mim install "mmdet==3.2.0"
mim install "mmpose>=1.0.0"
```

---

## 📂 Preparation

If you downloaded the full project from the Google Drive link above, these weights are already included. Otherwise, ensure you have the following:

1. **RTMDet Checkpoint**: `rtmdet_m_8xb32-100e_coco-obj365-person-235e8209.pth`
2. **RTMPose Checkpoint**: `rtmpose-x_simcc-body7_pt-body7_700e-384x288-71d7b7e9_20230629.pth`
3. **GAST-Net 3D Checkpoint**: `27_frame_model_causal.bin`

> 💡 **Tip:** Modify `mmpose_root`, `project_root`, and `input_video_path` under `if __name__ == "__main__":` to match your local file structure.

---

## 💻 Usage

Run the main script to start the real-time 3D motion capture pipeline:

```bash
python real-time2_opencv_halpe.py
```

* **📷 Camera Simulation**: Since no physical camera is connected for this demo, the system is configured to **simulate real-time camera input by reading from a video file**. The pipeline processes the video frames sequentially through the asynchronous buffer to mimic a live stream.
* **Real-time View**: A dual-screen window pops up (original video on the left, 3D skeleton on the right).
* **Exit**: Press `q` on your keyboard while focusing on the video window to safely terminate all threads.
* **Results**: The processed video is automatically saved to `./output/realtime_demo_output.mp4`.
