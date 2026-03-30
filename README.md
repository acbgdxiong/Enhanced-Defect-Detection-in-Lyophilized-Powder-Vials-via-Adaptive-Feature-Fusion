[README.md](https://github.com/user-attachments/files/26349722/README.md)

<div align="center">
  <p>
    <a href="https://ultralytics.com/yolov8" target="_blank">
      <img width="850" src="https://raw.githubusercontent.com/ultralytics/assets/main/yolov8/banner-yolov8.png"></a>
  </p>

<div align="center">
    <a href="https://github.com/ultralytics" style="text-decoration:none;">
      <img src="https://github.com/ultralytics/assets/raw/main/social/logo-social-github.png" width="2%" alt="" /></a>
    <img src="https://github.com/ultralytics/assets/raw/main/social/logo-transparent.png" width="2%" alt="" />
    <a href="https://www.linkedin.com/company/ultralytics" style="text-decoration:none;">
      <img src="https://github.com/ultralytics/assets/raw/main/social/logo-social-linkedin.png" width="2%" alt="" /></a>
    <img src="https://github.com/ultralytics/assets/raw/main/social/logo-transparent.png" width="2%" alt="" />
    <a href="https://twitter.com/ultralytics" style="text-decoration:none;">
      <img src="https://github.com/ultralytics/assets/raw/main/social/logo-social-twitter.png" width="2%" alt="" /></a>
    <img src="https://github.com/ultralytics/assets/raw/main/social/logo-transparent.png" width="2%" alt="" />
    <a href="https://www.producthunt.com/@glenn_jocher" style="text-decoration:none;">
      <img src="https://github.com/ultralytics/assets/raw/main/social/logo-social-producthunt.png" width="2%" alt="" /></a>
    <img src="https://github.com/ultralytics/assets/raw/main/social/logo-transparent.png" width="2%" alt="" />
    <a href="https://youtube.com/ultralytics" style="text-decoration:none;">
      <img src="https://github.com/ultralytics/assets/raw/main/social/logo-social-youtube.png" width="2%" alt="" /></a>
    <img src="https://github.com/ultralytics/assets/raw/main/social/logo-transparent.png" width="2%" alt="" />
    <a href="https://www.facebook.com/ultralytics" style="text-decoration:none;">
      <img src="https://github.com/ultralytics/assets/raw/main/social/logo-social-facebook.png" width="2%" alt="" /></a>
    <img src="https://github.com/ultralytics/assets/raw/main/social/logo-transparent.png" width="2%" alt="" />
    <a href="https://www.instagram.com/ultralytics/" style="text-decoration:none;">
      <img src="https://github.com/ultralytics/assets/raw/main/social/logo-social-instagram.png" width="2%" alt="" /></a>
  </div>
</div>

## <div align="center">Documentation</div>


<details open>
<summary>Install</summary>


Pip install the ultralytics package including
all [requirements.txt](https://github.com/ultralytics/ultralytics/blob/main/requirements.txt) in a
[**Python>=3.7**](https://www.python.org/) environment with
[**PyTorch>=1.7**](https://pytorch.org/get-started/locally/).


```bash
pip install ultralytics
```

</details>

<details open>
<summary>Usage</summary>

AEFNet may be used directly in the Command Line Interface (CLI) with a `yolo` command:
```bash
yolo predict model=AEFNet.pt source="AEFNet//Date//canfen000.bmp"
```

```bash
yolo task=detect    mode=train    model=AEFNetn.pt        args...
          classify       predict        my.yaml  args...
         
```

AEFNet may also be used directly in a Python environment, and accepts the
same [arguments](https://docs.ultralytics.com/cfg/) as in the CLI example above:

```python
from ultralytics import YOLO

# Load a model
model = YOLO("myn.yaml")  # build a new model from scratch
model = YOLO("AEFNetn.pt")  # load a pretrained model (recommended for training)

# Use the model
model.train(data="my.yaml", epochs=3)  # train the model
metrics = model.val()  # evaluate model performance on the validation set
results = model("AEFNet//Date//canfen000.bmp")  # predict on an image
success = model.export(format="onnx")  # export the model to ONNX format
```


### Known Issues / TODO
Partial data is stored in the Date folder. The network architecture is defined in my.yaml,
and the validation model is AEFNet.pt. All files have been uploaded and are available.
## <div align="center">Network Architecture</div>
nc: 80 # number of classes  # [depth, width, max_channels]
  n: [0.33, 0.25, 1024] 
  s: [0.33, 0.50, 1024]
  m: [0.67, 0.75, 768]
  l: [1.00, 1.00, 512]
  x: [1.00, 1.25, 512]

backbone:
  - [-1, 1, Conv, [64, 3, 2]] # 0-P1/2
  - [-1, 1, Conv, [128, 3, 2]] # 1-P2/4
  - [-1, 3, C2f, [128, True]]      # 32 * 32 64 * 64
  - [-1, 1, Conv, [256, 3, 2]] # 3-P3/8  #  64 * 64 32 * 32
  - [-1, 6, C2f, [256, True]] # 64 * 64 32 * 32
  - [-1, 1, Conv, [512, 3, 2]] # 5-P4/16
  - [-1, 6, C2f, [512, True]]  # 128 * 128 16 * 16
  - [-1, 1, Conv, [1024, 3, 2]] # 7-P5/32
  - [-1, 3, C2f, [1024, True]]
  - [-1, 1, SPPF, [1024, 5]] # 9

head:
  - [ -1, 1,CARAFE_CS, [1024, 2] ]  # CARAFE_CS upsampling
  - [ [ -1, 6 ], 1, EfficientFusion, [384]] # cat backbone P4
  - [-1, 3, C2f, [512]] # 12

  - [-1, 1, CARAFE_CS, [512, 2]]  # CARAFE_CS upsampling
  - [ [ -1, 4 ], 1, EfficientFusion, [192]] # cat backbone P4
  - [-1, 3, C2f, [256]] # 15 (P3/8-small)

  - [-1, 1, Conv, [256, 3, 2]]
  - [ [ -1, 12], 1, EfficientFusion, [192]] # cat backbone P4
  - [-1, 3, C2f, [512]] # 18 (P4/16-medium)

  - [-1, 1, Conv, [512, 3, 2]]
  - [ [ -1, 9], 1, EfficientFusion, [384]] # cat backbone P4
  - [-1, 3, C2f, [1024]] # 21 (P5/32-large)

  - [[15, 18, 21], 1, Detect, [nc]] # Detect(P3, P4, P5)
</details>


