# BEVFormer — Reproduction & Dataset Analysis  
**by Jossue Espinoza**

This repository aims to **reproduce [BEVFormer](https://github.com/fundamentalvision/BEVFormer/tree/master)(ECCV 2022)**. In addition, it documents **dataset creation and preparation** (nuScenes), including **label taxonomy**, **annotation format**, and the **data structure** required to train/evaluate the model.

---

## Default dataset used by BEVFormer
The official **BEVFormer repo** provides dataset preparation instructions in their docs [here](https://github.com/fundamentalvision/BEVFormer/blob/master/docs/prepare_dataset.md). 

  **NuScenes Dataset** -> [Download Here](https://www.nuscenes.org/download)
### 1. Download 
      - nuScenes V1.0 full dataset data
      - CAN bus expansion data (often distributed as can_bus.zip)
      - > [!NOTE] 
      - > Follow the official BEVFormer dataset doc to match exactly what the authors expect.
### 2. Extract and place CAN bus data
```bash
# Example: after downloading `can_bus.zip`
unzip can_bus.zip
# Move/merge the extracted folder into your nuScenes root directory.
# Keep the folder name exactly as nuScenes provides it.
mv can_bus /path/to/nuscenes/
```
> [!NOTE]
> If you plan to follow the pipeline with the recommended dataset, the cost of disk space is **373.3 GB**

### 3. Prepare nuScenes data
The original repo provides a script to create the requiered structure for the dataset, which genetate custom annotation files that are different from mmdet3d's, you can find it [here](https://github.com/fundamentalvision/BEVFormer/blob/master/tools/create_data.py). 

```bash
# To start the script
python tools/create_data.py nuscenes --root-path ./data/nuscenes --out-dir ./data/nuscenes --extra-tag nuscenes --version v1.0 --canbus ./data
```

**Supported datasets (directly convertible with `create_data.py`)**
- **`nuscenes`** — nuScenes v1.0 / v1.0-mini (includes temporal info + optional CAN bus, `max_sweeps`)
- **`kitti`** — KITTI (creates info `.pkl` files + 2D ann export + GT database)
- **`lyft`** — Lyft (creates info `.pkl` files; similar pipeline style to nuScenes)
- **`waymo`** — Waymo (converts to KITTI-format first, then creates info + GT database)
- **`scannet`** — ScanNet (indoor info generation)
- **`s3dis`** — S3DIS (indoor info generation)
- **`sunrgbd`** — SUN RGB-D (indoor info generation)

> [!IMPORTANT]
> - **BEVFormer is primarily designed for `nuscenes`** (multi-camera + temporal setup).
> - The other datasets are **supported by the converter script**, but may require **model/config changes** to be truly “BEVFormer-ready”.


## Create your OWN Dataseet
BEVFormer is commonly evaluated on **nuScenes** (multi-view, temporal, camera-only).  
If you want to train/evaluate on your *own* data:

> [!NOTE]
> This post cover the nuScenes dataset format, but if you see properly you can organice your data with any other format and then converted it with the script defined before

### Materials for Data Collection
This information was obteined from [nuSense-LidarSeg](https://www.nuscenes.org/nuscenes?tutorial=nuscenes)
- 6x camera (Basler acA1600-60gc):
- 1x spinning LIDAR (Velodyne HDL32E):
- 5x long range RADAR sensor (Continental ARS 408-21):
- GPS
- IMU
  
### Car Set Up
<p align="center">
  <img src="images / distribution.png" alt="Distribution of the Components" width="700">
  <br>
  <em>Distribution of the components - Image taked from nuScenes Documentation</em>
</p>  









