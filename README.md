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
  <img src="images/distribution.png" alt="Distribution of the Components" width="700">
  <br>
  <em> Figure 1. nuScenes sensor suite and coverage. Top: placement and coordinate axes of the vehicle-mounted sensors (6 cameras, 5 radars, LiDAR, and IMU).
</em>
</p>  

<p align="center">
  <img src="images/camera-fov.37c13013.jpg" alt="Distribution of the Components" width="400">
  <br>
  <em> Figure 2. approximate sensor fields of view around the vehicle. Source: nuScenes documentation.</em>
</p>  

### Sensor calibration

To achieve a high-quality multi-sensor dataset, it is essential to calibrate both **extrinsics** and **intrinsics** for every sensor. In this setup, **extrinsics are expressed relative to the ego frame** (defined as the midpoint of the rear vehicle axle). The main calibration steps are:

- **LiDAR extrinsics:** Use a laser line/liner to accurately measure the LiDAR position relative to the ego frame.  
- **Camera extrinsics:** Place a cube-shaped calibration target (three orthogonal planes with known patterns) in front of the camera and LiDAR. Detect the patterns, compute the camera-to-LiDAR transform by aligning the planes, then combine it with the LiDAR-to-ego transform to obtain the camera-to-ego extrinsics.  
- **Radar extrinsics:** Mount the radar horizontally and collect data while driving in an urban environment. Filter out radar returns from moving objects, then calibrate the yaw angle via brute-force search to minimize compensated range rates for static objects.  
- **Camera intrinsics:** Use a calibration board with known patterns to estimate camera intrinsic parameters and lens distortion.


### Sensor Synchronization

To achieve good cross-modality alignment between **LiDAR** and **cameras**, each camera exposure is **triggered when the top LiDAR sweeps across the center of the camera’s field of view (FOV)**. The **image timestamp** is the exposure trigger time, while the **LiDAR timestamp** corresponds to the time when the full rotation of the current LiDAR frame is completed. Since camera exposure is nearly instantaneous, this triggering strategy typically provides good alignment.

- **Camera rate:** 12 Hz  
- **LiDAR rate:** 20 Hz  
- The **12 camera exposures** are distributed as evenly as possible across the **20 LiDAR scans**, so **not every LiDAR scan has a corresponding camera frame**.  
- Using **12 Hz cameras** reduces **compute, bandwidth, and storage** requirements.

> **Non-official note (my suggestion):** for ROS2-based pipelines, a practical approach is to implement **custom synchronization messages / logic** for LiDAR–IMU (and extend similarly for cameras). Example implementation: [https://github.com/JossueE/FastLIO-ROS2-Simulation/blob/main/src/lidar_imu_sync.cpp](https://github.com/JossueE/FastLIO-ROS2-Simulation/blob/main/src/lidar_imu_sync.cpp)

### Data format

nuScenes stores its dataset information in a **structured relational schema** (a set of linked “tables”) that defines how **sensor data** (images, LiDAR, RADAR), **calibration**, **ego/vehicle poses**, **maps**, and **annotations** are organized and connected. Each record is identified by a unique **token**, and relationships between records are established through **foreign keys**, enabling consistent linking across time (scenes → samples) and across modalities (samples → sensor measurements → annotations).

This “data format” is important because it guarantees that every frame can be traced to:
- the exact sensor capture (with timestamps),
- the sensor geometry (intrinsics/extrinsics),
- the vehicle pose in the world,
- and the corresponding annotations.

> For the official and detailed description of each object/table and its fields, see:  
> [https://www.nuscenes.org/nuimages#tutorial](https://www.nuscenes.org/nuimages#tutorial)



<p align="center">
  <img src="images/schema.svg" alt="nuScenes Schema" width="900">
  <br>
  <em> Figure 2. nuScenes Schema</em>
</p>  


## Data Annotation (nuScenes-style) — A faithful, step-by-step workflow

This section documents a **practical annotation workflow** aligned with the official nuScenes description: **scene selection → 2 Hz keyframes → continuous 3D cuboid annotation with LiDAR/RADAR coverage → multiple validation passes → export in a linked database format**. 

After adquire the data, have a proper sensor calibration and Synchronization we:

### Select “scenes” (clips) to annotate

nuScenes is composed of **1000 scenes**, each **20 seconds long**, chosen to emphasize diversity and challenging situations.  
They also include **textual scene descriptions** written by expert annotators.

**Reproduction checklist for an own dataset:**
- Keep a consistent scene duration (e.g., 20s) so temporal statistics match.
- Store a short text caption/description per scene (optional but useful).

---

### Sample keyframes at 2 Hz (the frames that receive GT labels)

After scenes are selected, nuScenes **samples keyframes at 2 Hz** (image/lidar/radar keyframes). 
That implies ~**40 labeled keyframes per 20s scene** (20s × 2 Hz), while intermediate sensor frames can still be kept for tracking/prediction. 

**Key point:** decide and lock **keyframe timestamps**, because the rest of annotation is indexed around them.

---

### Annotate every keyframe (what each label must contain)

For **each keyframe**, nuScenes annotates **all 23 object classes** with: 

1) **Semantic category**  
2) **Attributes** (they describe attributes such as **visibility, activity, and pose**; nuScenes overall includes **8 attributes**)
3) **3D cuboid** parameterized as:  
   **(x, y, z, width, length, height, yaw)** 

#### The 23-class taxonomy (as referenced in the nuScenes paper)
The nuScenes paper describes 23 general classes and shows them in a mapping table (prefixes omitted in the table for brevity). 

- animal  
- debris  
- pushable_pullable  
- bicycle_rack  
- ambulance  
- police  
- barrier  
- bicycle  
- bus.bendy  
- bus.rigid  
- car  
- construction  
- motorcycle  
- adult  
- child  
- construction_worker  
- police_officer  
- personal_mobility  
- stroller  
- wheelchair  
- trafficcone  
- trailer  
- truck 

> For the **exact official naming (including prefixes)** and the authoritative taxonomy objects, consult the official devkit/tutorial references:
> - https://www.nuscenes.org/nuscenes?tutorial=nuscenes  
> - https://github.com/nutonomy/nuscenes-devkit/blob/master/docs/instructions_nuscenes.md  
> The nuScenes paper explicitly points to the devkit as the public source for **taxonomy + annotation instructions**.

---

### References (official)
- nuScenes paper (dataset design + annotation workflow + calibration/sync): https://ar5iv.labs.arxiv.org/html/1903.11027  
- nuScenes tutorial: https://www.nuscenes.org/nuscenes?tutorial=nuscenes  
- nuScenes devkit annotation instructions: https://github.com/nutonomy/nuscenes-devkit/blob/master/docs/instructions_nuscenes.md

## Code Example
[nuScenes tutorial](https://www.nuscenes.org/tutorials/nuscenes_tutorial.html)







