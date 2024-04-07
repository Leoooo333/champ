# How to produce Motion

Try Champ with your dance videos! It may take time to setup the environment, follow the instruction step by step🐢, report issue when necessary.


## Install dependencies

1. Install PyPI dependencies

    ```shell
    pip install -r requirements.txt
    ```
2. Install gcc and g++ in conda environment

    gcc and g++ 12 is necessary to build detectron2

    ```shell
    conda install -c conda-forge gcc=12 gxx=12
    ```

3. Install [4D-Humans](https://github.com/shubham-goel/4D-Humans)
    ```shell
    git clone https://github.com/shubham-goel/4D-Humans.git
    
    pip install -e 4D-Humans
    ```

    or you can install via pip by a simple command
    ```shell
    pip install git+https://github.com/shubham-goel/4D-Humans
    ```

4. Install [detectron2](https://github.com/facebookresearch/detectron2)

    ```shell
    git clone https://github.com/facebookresearch/detectron2

    pip install -e detectron2
    ```
    or you can install via pip by a simple command
    ```shell
    pip install git+https://github.com/facebookresearch/detectron2
    ```

5. Install [Blender](https://www.blender.org/)

    You can download Blender 3.x version for your operation system from this url [https://download.blender.org/release/Blender3.6](https://download.blender.org/release/Blender3.6/).

## Download models

1. [DWPose for controlnet](https://github.com/IDEA-Research/DWPose?tab=readme-ov-file#-dwpose-for-controlnet)

    First, you need to download our Pose model dw-ll_ucoco_384.onnx ([baidu](https://pan.baidu.com/s/1nuBjw-KKSxD_BkpmwXUJiw?pwd=28d7), [google](https://drive.google.com/file/d/12L8E2oAgZy4VACGSK9RaZBZrfgx7VTA2/view?usp=sharing)) and Det model yolox_l.onnx ([baidu](https://pan.baidu.com/s/1fpfIVpv5ypo4c1bUlzkMYQ?pwd=mjdn), [google](https://drive.google.com/file/d/1w9pXC8tT0p9ndMN-CArp1__b2GbzewWI/view)), then put them into `pretrained_models/DWPose/`.


2. HMR2 checkpoints

    ```shell
    python -m scripts.pretrained_models.download --hmr2 true
    ```
3. Detectron2 model

    ```shell
    python -m scripts.pretrained_models.download --detectron2 true
    ```
4. SMPL model

    You need to download smpl model manually from https://smplify.is.tue.mpg.de/ and place it at pretrain_models/smpl/SMPL_NEUTRAL.pkl



## Produce motion data


1. Prepare video

    Prepare a "dancing" video, and use `ffmpeg` to split it into frame images:
    ```shell
    ffmpeg -i your_video_file.mp4 -c:v png driving_videos/Video_1/images/%04d.png
    ```

2. Inference SMPL

    Make sure you have splitted the video into frames and organized the image files as below:
    ```
    |--video_folder
        |--images
            |--0001.png
            |--0002.png
            |--0003.png
            |--0004.png
            |--0005.png
            ......
    ```

    Then run smpl inference script to produce smpl:

    ```shell
    python -m scripts.data_processors.smpl.inference --driving_video_path driving_videos/Video_1 --device 0
    ```

    All SMPL results will appear under `driving_videos/Video_1`.


3. Smooth SMPL (optional)

    **TODO:** Coming Soon.

4. Transfer SMPL

    ```shell
    python -m scripts.data_processors.smpl.transfer --reference_path example_data/test_imgs/smpl_results/ref-01.npy --driving_path driving_videos/Video_1 --output_folder results/smpl/transfered --figure_transfer --view_transfer

    ```

5. Render SMPL via Blender

    ```shell
    blender scripts/data_processors/smpl/blend/smpl_rendering.blend --background --python scripts/data_processors/smpl/render.py --driving_path driving_videos/Video_1/smpl_results --reference_path example_data/test_imgs/images/ref-01.png
    ```

6. Inference DWPose

    ```shell
    python -m scripts.data_processors.dwpose.inference --input /home/leeway/workspace/champ/github/driving_videos/Video_1/images --cpu True --output ./driving_videos/Video_1/dwpose
    ```
