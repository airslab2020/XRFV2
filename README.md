# XRFv2 Plus Update

**Update time**: June 6, 2026 (CST). The public Kaggle release status was last verified through version 4 on June 5, 2026, 23:39 CST.

This section summarizes the **XRFv2 Plus** release, a video-aligned multimodal update to XRFV2. The original XRFV2 README is kept unchanged below the separator.

**Kaggle release**: https://www.kaggle.com/datasets/airslab2020/xrfv2-multimodal-tal-caption-qa-no-rgb

**Preprint**: Fei Wang, *XRFv2 Plus: A Multimodal Sensor-Vision-Language Dataset for Action Understanding*. Zenodo, June 6, 2026. https://doi.org/10.5281/zenodo.20564312

**BibTeX**:

```bibtex
@misc{wang2026xrfv2plus,
  author       = {Wang, Fei},
  title        = {XRFv2 Plus: A Multimodal Sensor-Vision-Language Dataset for Action Understanding},
  year         = {2026},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.20564312},
  url          = {https://doi.org/10.5281/zenodo.20564312}
}
```

**Update note**: *XRFv2 Plus: A Multimodal Sensor-Vision-Language Dataset for Action Understanding*

The Kaggle release is now public. The current public Kaggle dataset is at version 4: version 1 provides the fast no-video release, version 2 adds the 853 cropped Kinect depth videos, version 3 adds the 853 cropped Kinect infrared videos, and version 4 adds the DensePose H5 file.

## What Is New

XRFv2 Plus converts the original XRFV2 recordings into a synchronized action-understanding release. Each updated sample is aligned to the cropped Kinect video timeline and uses relative time as the common temporal axis.

The update adds synchronized sensing streams, body information, and language/action-understanding annotations for the same **853 valid action sequences**.

| Category | New release content |
| --- | --- |
| Sensing streams | WiFi CSI, five-position IMU, AirPods IMU, RGB video embeddings, Kinect depth videos, Kinect infrared videos |
| Body information | Human 2D pose, depth-assisted 3D pose, SMPL mesh, DensePose-style surface information |
| Action labels | Temporal action localization, action captioning, action question answering |
| Time convention | All labels and synchronized tensors use relative time within each sequence |

## Release Summary

| Item | Value |
| --- | --- |
| Standardized samples | 853 |
| Original candidates | 880 |
| Excluded candidates | 27 with incomplete five-position IMU coverage |
| Volunteers | 16 |
| Scenes | Diningroom, Studyroom, bedroom |
| Action classes | 34 |
| Action segments | 8157 |
| Sequence duration range | 43-89 seconds |
| Actions per sequence | 7-11 |
| Total video-aligned duration | 16 h 14 min 38 s |

## Main Files and Tensor Conventions

For a sequence of duration `T` seconds:

| Modality | Sampling / stride | Shape or file convention | File |
| --- | --- | --- | --- |
| WiFi CSI amplitude/phase | 50 Hz | `[50*T, 3, 3, 30]` | `wifi_50hz_853_video_aligned.h5` |
| Five-position IMU | 50 Hz | `[50*T, 5, 6]` | `imu_50hz_853_video_aligned.h5` |
| AirPods Pro IMU | 25 Hz | `[25*T, 6]` | `airpods_25hz_853_video_aligned.h5` |
| RGB video feature | 1 vector/s | `[T, 1024]` | `video_feature_853_video_aligned.h5` |
| 2D pose | frame-aligned | COCO-17 keypoints | `pose2d_coco17_yolov8m_853_video_aligned.h5` |
| 3D pose | frame-aligned | COCO-17 depth-camera keypoints | `pose3d_coco17_depth_853_video_aligned.h5` |
| Human mesh | frame-aligned | compact ROMP/SMPL parameters | `human_mesh_smpl_853_video_aligned.h5` |
| DensePose | frame-aligned | compressed DensePose IUV information | `densepose_rcnn_R50_FPN_853_video_aligned.h5` |
| Kinect depth videos | 15 FPS, 512 x 512 | `depth_cropped.mkv` | `videos/kinect_rgb_ir_depth/<scene>/<sample>/` |
| Kinect infrared videos | 15 FPS, 512 x 512 | `ir_cropped.mkv` | `videos/kinect_rgb_ir_depth/<scene>/<sample>/` |

The five-position IMU order is fixed as:

| Index | Position |
| --- | --- |
| 0 | left wrist |
| 1 | right wrist |
| 2 | phone case in the left pants pocket |
| 3 | phone case in the right pants pocket, also used as the phone surrogate for phone-related actions |
| 4 | glasses temple |

## Preprocessing Notes

- RGB video features are extracted from cropped Kinect RGB videos using TorchVision `swin3d_b` with `Swin3D_B_Weights.KINETICS400_IMAGENET22K_V1`. The classifier head is replaced by `torch.nn.Identity()`, giving one 1024-D vector per second.
- WiFi, five-position IMU, and AirPods streams are segmented to the video-aligned action window. IMU and AirPods are resampled **per second** rather than by stretching the whole sequence, so integer second boundaries stay aligned with video features and TAL labels.
- Cropped depth and infrared videos are kept at 15 FPS with 512 x 512 resolution.
- If a human detector returns multiple people, the single most likely person is retained because the XRFv2 scenes contain one acting participant.
- A small set of video-length corrections was handled during alignment: five samples used tail padding for RGB features, and seven bedroom samples removed the last one or two actions because the cropped RGB video did not cover the complete command-time sequence.

## Annotation Tasks

| Annotation | Format / content |
| --- | --- |
| Temporal action localization | ActivityNet-style JSON with relative-time segments, normalized segments, action IDs, scene IDs, volunteer IDs, and sequence IDs |
| Action captioning | Chinese-only, English-only, and bilingual files with dense segment captions and one sequence-level caption per sample |
| Action QA | Chinese-only, English-only, and bilingual QA files with 108,929 QA pairs across 21 question types |

Example sequence-level caption:

> The participant gets out of bed, then stands up, then stretches, then walks to the window, then opens or closes the curtains, then opens or closes the window, then waters the plant, then walks to the bed, then sits down, and finally lies down.

Example Action QA items:

| Type | Question | Answer |
| --- | --- | --- |
| Existence | Did the participant sit down? | yes |
| Counting | How many times did the participant sit down? | 1 |
| Segment duration | How long did the first action, sitting down, last? | 6 seconds |
| Temporal order | Does sitting down happen before walking toward the chair? | yes |
| Time-point action | What action was the participant doing at 3 seconds? | sitting down |

## Kaggle Upload Batches

Status below was last checked on **June 5, 2026, 23:39 CST**. The exact display time on Kaggle may lag behind upload completion because Kaggle needs to finalize each dataset version.

| Batch | Content | Current status | Estimated availability (CST) |
| --- | --- | --- | --- |
| Fast no-video release | annotations, WiFi, IMU, AirPods, RGB features, 2D pose, 3D pose, SMPL mesh | public; Kaggle version 1 is Ready | available now |
| Depth video batch | 853 cropped Kinect depth videos, about 57.63 GiB | public; Kaggle version 2 is Ready | available now |
| Infrared video batch | 853 cropped Kinect infrared videos, about 94.02 GiB | public; Kaggle version 3 is Ready | available now |
| DensePose batch | DensePose H5, about 37.5 GiB | public; Kaggle version 4 is Ready | available now |

Raw Kinect recordings and cropped Kinect RGB videos are not included in the Kaggle release because of privacy considerations.

---

# XRFV2

<p align="center">
  <img src="img/story.png" alt="image-20240719171906628" width="700px"/>
</p>


**XRF V2: A Dataset for Action Summarization with Wi-Fi Signals, and IMUs in Phones, Watches, Earbuds, and Glasses**

XRF V2 is a dataset designed for action summarization tasks using Wi-Fi signals and IMUs data from various devices such as phones, watches, earbuds, and glasses. This dataset provides valuable insights into human activity recognition and summarization using multi-modal sensor data.

**📊 Download Link**: 
-   Kaggle (IMU and Wi-Fi: ): https://www.kaggle.com/datasets/anonymous20251/xrfv2dataset
-   SDP  (IMU, Wi-Fi): http://www.sdp8.org/Dataset?id=1186880c-b321-45d0-ac3a-74ef9d2fdeda

- Models' weights: https://drive.google.com/drive/folders/1N3Ytqp0UjiBdSc_rb3kPjjwejmdtZEK1?usp=sharing




## 📦 Environment Configuration

### 🛠️ Mamba Environment Setup:
Ensure that you are using the **CUDA 11.8** environment.

```bash
# Clone the video-mamba-suite repository
git clone --recursive https://github.com/OpenGVLab/video-mamba-suite.git

# Create and activate the environment
conda create -n video-mamba-suite python=3.9
conda activate video-mamba-suite

# Install PyTorch
pip install torch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 --index-url https://download.pytorch.org/whl/cu118

# Install required dependencies
pip install h5py pandas scipy torchinfo

# Install the requirements from requirement.txt
pip install -r requirement.txt

# Install causal-conv1d
cd causal-conv1d
# If setup.py fails, run the following:
CAUSAL_CONV1D_FORCE_BUILD=TRUE pip install .
cd ..

# Install mamba
cd mamba
python setup.py develop
cd ..
```
⚠️ If you encounter issues while installing `causal-conv1d`, please refer to [this setup issue fix](https://github.com/state-spaces/mamba/issues/40#issuecomment-1849095898).

## 🏃‍♂️ Running the Code:

1. Modify the paths in `basic_config.json` to match your system setup.
2. To **train** the model:
```bash
   python script/train_run.py
```
3. To **test** the model:

Copy the path of the trained model and specify it in `test_run.py` before running the test:

```
   test_model_list = [XXXXX]
```

```bash
   python script/test_run.py
```
## 📞 Support
If you encounter any issues or need assistance, feel free to reach out to us.

## 📝 TODO
- To process video into 2D pose, 3D pose, and mesh for pose estimation and tracking, mesh reconstruction and tracking.
- To process video into internvideo6b features for multimodal learning.


## License 📜
XRFV2 is licensed under the MIT License. See the LICENSE file for more details.

## Citation
If XRFV2 helps in your research, please kindly cite 
```
@article{lan2025xrf,
  author = {Lan, Bo and Li, Pei and Yin, Jiaxi and Song, Yunpeng and Wang, Ge and Ding, Han and Han, Jinsong and Wang, Fei},
  title = {XRF V2: A Dataset for Action Summarization with Wi-Fi Signals, and IMUs in Phones, Watches, Earbuds, and Glasses},
  journal = {Proceedings of the ACM on Interactive, Mobile, Wearable and Ubiquitous Technologies},
  volume={9},
  number={3},
  pages={1--41},
  year = {2025},
  publisher = {ACM}
}
```
