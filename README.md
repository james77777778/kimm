<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD041 -->

<div align="center">
<img width="50%" src="https://github.com/james77777778/kimm/assets/20734616/b21db8f2-307b-4791-b93d-e913e45fb238" alt="KIMM">

[![Keras](https://img.shields.io/badge/keras-v3.3.0+-success.svg)](https://github.com/keras-team/keras)
[![PyPI](https://img.shields.io/pypi/v/kimm)](https://pypi.org/project/kimm/)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/james77777778/kimm/issues)
[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/james77777778/keras-image-models/actions.yml?label=tests)](https://github.com/james77777778/keras-image-models/actions/workflows/actions.yml?query=branch%3Amain++)
[![codecov](https://codecov.io/gh/james77777778/keras-image-models/graph/badge.svg?token=eEha1SR80D)](https://codecov.io/gh/james77777778/keras-image-models)
</div>

# Keras Image Models

- [Latest Updates](#latest-updates)
- [Introduction](#introduction)
- [Usage](#usage)
- [Installation](#installation)
- [Quickstart](#quickstart)
  - [Image classification with ImageNet weights](#image-classification-using-the-model-pretrained-on-imagenet)
  - [An end-to-end fine-tuning example: cats vs. dogs dataset](#an-end-to-end-example-fine-tuning-an-image-classification-model-on-a-cats-vs-dogs-dataset)
  - [Grad-CAM](#grad-cam)
- [Model Zoo](#model-zoo)
- [License](#license)
- [Acknowledgements](#acknowledgements)

## Latest Updates

2024/06/02:

- Add docstrings for all `kimm` models.
- Merge reparameterizable layers into 1 `ReparameterizableConv2D`
- Add `GhostNetV3*` from [huawei-noah/Efficient-AI-Backbones](https://github.com/huawei-noah/Efficient-AI-Backbones)

## Introduction

**K**eras **Im**age **M**odels (`kimm`) is a collection of image models, blocks and layers written in Keras 3. The goal is to offer SOTA models with pretrained weights in a user-friendly manner.

**KIMM** is:

- 🚀 A model zoo where almost all models come with **pre-trained weights on ImageNet**.
- 🧰 Providing APIs to export models to `.tflite` and `.onnx`.
- 🔧 Supporting the **reparameterization** technique.
- ✨ Integrated with **feature extraction** capability.

## Usage

- `kimm.list_models`
- `kimm.models.*.available_feature_keys`
- `kimm.models.*(...)`
- `kimm.models.*(..., feature_extractor=True, feature_keys=[...])`

```python
import keras
import kimm

# List available models
print(kimm.list_models("mobileone", weights="imagenet"))
# ['MobileOneS0', 'MobileOneS1', 'MobileOneS2', 'MobileOneS3']

# Initialize model with pretrained ImageNet weights
# Note: all `kimm` models expect inputs in the value range of [0, 255] by
# default if `include_preprocessing=True`
x = keras.random.uniform([1, 224, 224, 3]) * 255.0
model = kimm.models.MobileOneS0()
y = model.predict(x)
print(y.shape)
# (1, 1000)

# Print some basic information about the model
print(model)
# <MobileOneS0 name=MobileOneS0, input_shape=(None, None, None, 3),
# default_size=224, preprocessing_mode="imagenet", feature_extractor=False,
# feature_keys=None>
# This information can also be accessed through properties
print(model.input_shape, model.default_size, model.preprocessing_mode)

# List available feature keys of the model class
print(kimm.models.MobileOneS0.available_feature_keys)
# ['STEM_S2', 'BLOCK0_S4', 'BLOCK1_S8', 'BLOCK2_S16', 'BLOCK3_S32']

# Enable feature extraction by setting `feature_extractor=True`
# `feature_keys` can be optionally specified
feature_extractor = kimm.models.MobileOneS0(
    feature_extractor=True, feature_keys=["BLOCK2_S16", "BLOCK3_S32"]
)
features = feature_extractor.predict(x)
for feature_name, feature in features.items():
    print(feature_name, feature.shape)
# BLOCK2_S16 (1, 14, 14, 256), BLOCK3_S32 (1, 7, 7, 1024), ...
```

> [!NOTE]  
> All models in `kimm` expect inputs in the value range of [0, 255] by default if `include_preprocessing=True`.
> Some models only accept static inputs. You should explicitly specify the input shape for these models by `input_shape=[*, *, 3]`.

## Advanced Usage

- `kimm.utils.get_reparameterized_model`
- `kimm.export.export_tflite`
- `kimm.export.export_onnx`

```python
import keras
import kimm
import numpy as np

# Initialize a reparameterizable model
x = keras.random.uniform([1, 224, 224, 3]) * 255.0
model = kimm.models.MobileOneS0()
y = model.predict(x)

# Get reparameterized model by kimm.utils.get_reparameterized_model
reparameterized_model = kimm.utils.get_reparameterized_model(model)
y2 = reparameterized_model.predict(x)
np.testing.assert_allclose(
    keras.ops.convert_to_numpy(y), keras.ops.convert_to_numpy(y2), atol=1e-3
)

# Export model to tflite format
kimm.export.export_tflite(reparameterized_model, 224, "model.tflite")

# Export model to onnx format
# Note: must be "channels_first" format before the exporting
# kimm.export.export_onnx(reparameterized_model, 224, "model.onnx")
```

## Installation

```bash
pip install keras kimm -U
```

> [!IMPORTANT]  
> Make sure you have installed a supported backend for Keras.

## Quickstart

### Image classification using the model pretrained on ImageNet

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/14WxYgVjlwCIO9MwqPYW-dskbTL2UHsVN?usp=sharing)

Using `kimm.models.VisionTransformerTiny16`:

<div align="center">
<img width="50%" src="https://github.com/james77777778/keras-image-models/assets/20734616/7caa4e5e-8561-425b-aaf2-6ae44ac3ea00" alt="african_elephant">
</div>

```bash
1/1 ━━━━━━━━━━━━━━━━━━━━ 1s 1s/step
Predicted: [('n02504458', 'African_elephant', 0.6895825), ('n01871265', 'tusker', 0.17934209), ('n02504013', 'Indian_elephant', 0.12927249)]
```

### An end-to-end example: fine-tuning an image classification model on a cats vs. dogs dataset

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1IbqfqG2NKEOKvBOznIPT1kjOdVPfThmd?usp=sharing)

Using `kimm.models.EfficientNetLiteB0`:

<div align="center">
<img width="75%" src="https://github.com/james77777778/kimm/assets/20734616/cbfc0773-a3fa-407d-be9a-fba4f19da6d3" alt="kimm_prediction_0">

<img width="75%" src="https://github.com/james77777778/kimm/assets/20734616/2eac0831-75bb-4790-a3af-412c3e09cf8f" alt="kimm_prediction_1">
</div>

Reference: [Transfer learning & fine-tuning (keras.io)](https://keras.io/guides/transfer_learning/#an-endtoend-example-finetuning-an-image-classification-model-on-a-cats-vs-dogs-dataset)

### Grad-CAM

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1h25VmsYDOLL6BNbRPEVOh1arIgcEoHu6?usp=sharing)

Using `kimm.models.MobileViTS`:

<div align="center">
<img width="50%" src="https://github.com/james77777778/kimm/assets/20734616/cb5022a3-aaea-4324-a9cd-3d2e63a0a6b2" alt="grad_cam">
</div>

Reference: [Grad-CAM class activation visualization (keras.io)](https://keras.io/examples/vision/grad_cam/)

## Model Zoo

|Model|Paper|Weights are ported from|API (`kimm.models.*`)|
|-|-|-|-|
|ConvMixer|[ICLR 2022 Submission](https://arxiv.org/abs/2201.09792)|`timm`|`ConvMixer*`|
|ConvNeXt|[CVPR 2022](https://arxiv.org/abs/2201.03545)|`timm`|`ConvNeXt*`|
|DenseNet|[CVPR 2017](https://arxiv.org/abs/1608.06993)|`timm`|`DenseNet*`|
|EfficientNet|[ICML 2019](https://arxiv.org/abs/1905.11946)|`timm`|`EfficientNet*`|
|EfficientNetLite|[ICML 2019](https://arxiv.org/abs/1905.11946)|`timm`|`EfficientNetLite*`|
|EfficientNetV2|[ICML 2021](https://arxiv.org/abs/2104.00298)|`timm`|`EfficientNetV2*`|
|GhostNet|[CVPR 2020](https://arxiv.org/abs/1911.11907)|`timm`|`GhostNet*`|
|GhostNetV2|[NeurIPS 2022](https://arxiv.org/abs/2211.12905)|`timm`|`GhostNetV2*`|
|GhostNetV3|[arXiv 2024](https://arxiv.org/abs/2404.11202)|`github`|`GhostNetV3*`|
|HGNet||`timm`|`HGNet*`|
|HGNetV2||`timm`|`HGNetV2*`|
|InceptionNeXt|[CVPR 2024](https://arxiv.org/abs/2303.16900)|`timm`|`InceptionNeXt*`|
|InceptionV3|[CVPR 2016](https://arxiv.org/abs/1512.00567)|`timm`|`InceptionV3`|
|LCNet|[arXiv 2021](https://arxiv.org/abs/2109.15099)|`timm`|`LCNet*`|
|MobileNetV2|[CVPR 2018](https://arxiv.org/abs/1801.04381)|`timm`|`MobileNetV2*`|
|MobileNetV3|[ICCV 2019](https://arxiv.org/abs/1905.02244)|`timm`|`MobileNetV3*`|
|MobileOne|[CVPR 2023](https://arxiv.org/abs/2206.04040)|`timm`|`MobileOne*`|
|MobileViT|[ICLR 2022](https://arxiv.org/abs/2110.02178)|`timm`|`MobileViT*`|
|MobileViTV2|[arXiv 2022](https://arxiv.org/abs/2206.02680)|`timm`|`MobileViTV2*`|
|RegNet|[CVPR 2020](https://arxiv.org/abs/2003.13678)|`timm`|`RegNet*`|
|RepVGG|[CVPR 2021](https://arxiv.org/abs/2101.03697)|`timm`|`RepVGG*`|
|ResNet|[CVPR 2015](https://arxiv.org/abs/1512.03385)|`timm`|`ResNet*`|
|TinyNet|[NeurIPS 2020](https://arxiv.org/abs/2010.14819)|`timm`|`TinyNet*`|
|VGG|[ICLR 2015](https://arxiv.org/abs/1409.1556)|`timm`|`VGG*`|
|ViT|[ICLR 2021](https://arxiv.org/abs/2010.11929)|`timm`|`VisionTransformer*`|
|Xception|[CVPR 2017](https://arxiv.org/abs/1610.02357)|`keras`|`Xception`|

The export scripts can be found in `tools/convert_*.py`.

## License

Please refer to [timm](https://github.com/huggingface/pytorch-image-models#licenses) as this project is built upon it.

### `kimm` Code

The code here is licensed Apache 2.0.

## Acknowledgements

Thanks for these awesome projects that were used in `kimm`

- [https://github.com/keras-team/keras](https://github.com/keras-team/keras)
- [https://github.com/huggingface/pytorch-image-models](https://github.com/huggingface/pytorch-image-models)

## Citing

### BibTeX

```bash
@misc{rw2019timm,
  author = {Ross Wightman},
  title = {PyTorch Image Models},
  year = {2019},
  publisher = {GitHub},
  journal = {GitHub repository},
  doi = {10.5281/zenodo.4414861},
  howpublished = {\url{https://github.com/rwightman/pytorch-image-models}}
}
```

```bash
@misc{hy2024kimm,
  author = {Hongyu Chiu},
  title = {Keras Image Models},
  year = {2024},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/james77777778/kimm}}
}
```
