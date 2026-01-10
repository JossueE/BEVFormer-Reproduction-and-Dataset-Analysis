# BEVFormer — Reproduction & Dataset Analysis  
**by Jossue Espinoza**

This repository aims to **reproduce [BEVFormer](https://github.com/fundamentalvision/BEVFormer/tree/master)(ECCV 2022)**. In addition, it documents **dataset creation and preparation** (nuScenes), including **label taxonomy**, **annotation format**, and the **data structure** required to train/evaluate the model.

---

## Default dataset used by BEVFormer
The official **BEVFormer repo** provides dataset preparation instructions in their docs [here](https://github.com/fundamentalvision/BEVFormer/blob/master/docs/prepare_dataset.md). 

  **NuScenes Dataset** -> [Download Here](https://www.nuscenes.org/download)
    - 1. Download 
      - nuScenes V1.0 full dataset data
      - CAN bus expansion data (often distributed as can_bus.zip)
      - > [!NOTE] 
      - > Follow the official BEVFormer dataset doc to match exactly what the authors expect.
    - 2. Extract and place CAN bus data
    ''' bash
    # Example: after downloading `can_bus.zip`
    unzip can_bus.zip
    # Move/merge the extracted folder into your nuScenes root directory.
    # Keep the folder name exactly as nuScenes provides it.
    mv can_bus /path/to/nuscenes/
    '''
