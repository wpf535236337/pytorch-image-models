# PyTorch Image Models, etc

## Introduction 

For each competition, personal, or freelance project involving images + Convolution Neural Networks, I build on top of an evolving collection of code and models. This repo contains a (somewhat) cleaned up and paired down iteration of that code. Hopefully it'll be of use to others.

The work of many others is present here. I've tried to make sure all source material is acknowledged:
* Training/validation scripts evolved from early versions of the [PyTorch Imagenet Examples](https://github.com/pytorch/examples)
* CUDA specific performance enhancements have been pulled from [NVIDIA's APEX Examples](https://github.com/NVIDIA/apex/tree/master/examples)
* Models are from a wide variety of sources
    * [Torchvision](https://github.com/pytorch/vision/tree/master/torchvision/models)
    * [Cadene's Pretrained Models](https://github.com/Cadene/pretrained-models.pytorch)
    * [Myself](https://github.com/rwightman/pytorch-dpn-pretrained)
* LR scheduler ideas from [AllenNLP](https://github.com/allenai/allennlp/tree/master/allennlp/training/learning_rate_schedulers) and [FAIRseq](https://github.com/pytorch/fairseq/tree/master/fairseq/optim/lr_scheduler)
* Random Erasing from [Zhun Zhong](https://github.com/zhunzhong07/Random-Erasing/blob/master/transforms.py)

## Models

I've included a few of my favourite models, but this is not an exhaustive collection. You can't do better than Cadene's collection in that regard. Most models do have pretrained weights from their respective sources or original authors. 

* ResNet/ResNeXt (from [torchvision](https://github.com/pytorch/vision/tree/master/torchvision/models) with ResNeXt mods by myself)
    * ResNet-18, ResNet-34, ResNet-50, ResNet-101, ResNet-152, ResNeXt50 (32x4d), ResNeXt101 (32x4d and 64x4d)
* DenseNet (from [torchvision](https://github.com/pytorch/vision/tree/master/torchvision/models))
    * DenseNet-121, DenseNet-169, DenseNet-201, DenseNet-161
* Squeeze-and-Excitation ResNet/ResNeXt (from [Cadene](https://github.com/Cadene/pretrained-models.pytorch) with some pretrained weight additions by myself)
    * SENet-154, SE-ResNet-18, SE-ResNet-34, SE-ResNet-50, SE-ResNet-101, SE-ResNet-152, SE-ResNeXt-26 (32x4d), SE-ResNeXt50 (32x4d), ResNeXt101 (32x4d)
* Inception-ResNet-V2 and Inception-V4 (from [Cadene](https://github.com/Cadene/pretrained-models.pytorch) )
* Xception (from [Cadene](https://github.com/Cadene/pretrained-models.pytorch))
* PNasNet (from [Cadene](https://github.com/Cadene/pretrained-models.pytorch))
* DPN (from [me](https://github.com/rwightman/pytorch-dpn-pretrained), weights hosted by Cadene)
    * DPN-68, DPN-68b, DPN-92, DPN-98, DPN-131, DPN-107
* Generic MobileNet (from my standalone [GenMobileNet](https://github.com/rwightman/genmobilenet-pytorch)) - A generic model that implements many of the mobile optimized architecture search derived models that utilize similar DepthwiseSeparable and InvertedResidual blocks
    * MNASNet B1, A1 (Squeeze-Excite), and Small
    * MobileNet-V1
    * MobileNet-V2
    * MobileNet-V3 (work in progress, validating config)
    * ChamNet (details hard to find, currently an educated guess)
    * FBNet-C (TODO A/B variants)
## Features
Several (less common) features that I often utilize in my projects are included. Many of their additions are the reason why I maintain my own set of models, instead of using others' via PIP:
* All models have a common default configuration interface and API for
    * accessing/changing the classifier - `get_classifier` and `reset_classifier`
    * doing a forward pass on just the features - `forward_features`
    * these makes it easy to write consistent network wrappers that work with any of the models
* All models have a consistent pretrained weight loader that adapts last linear if necessary, and from 3 to 1 channel input if desired
* The train script works in several process/GPU modes:
    * NVIDIA DDP w/ a single GPU per process, multiple processes with APEX present (AMP mixed-precision optional)
    * PyTorch DistributedDataParallel w/ multi-gpu, single process (AMP disabled as it crashes when enabled)
    * PyTorch w/ single GPU single process (AMP optional)
* A dynamic global pool implementation that allows selecting from average pooling, max pooling, average + max, or concat([average, max]) at model creation. All global pooling is adaptive average by default and compatible with pretrained weights.
* A 'Test Time Pool' wrapper that can wrap any of the included models and usually provide improved performance doing inference with input images larger than the training size. Idea adapted from original DPN implementation when I ported (https://github.com/cypw/DPNs)
* Training schedules and techniques that provide competitive results (Cosine LR, Random Erasing, Label Smoothing, etc)
* Mixup (as in https://arxiv.org/abs/1710.09412) - currently implementing/testing
* An inference script that dumps output to CSV is provided as an example

### Custom Weights
I've leveraged the training scripts in this repository to train a few of the models with missing weights to good levels of performance. These numbers are all for 224x224 training and validation image sizing with the usual 87.5% validation crop.

|Model | Prec@1 (Err) | Prec@5 (Err) | Param # | Image Scaling  |
|---|---|---|---|---|
| ResNeXt-50 (32x4d) | 78.512 (21.488) | 94.042 (5.958) | 25M | bicubic |
| SE-ResNeXt-26 (32x4d) | 77.104 (22.896) | 93.316 (6.684) | 16.8M | bicubic |
| SE-ResNet-34 | 74.808 (25.192) | 92.124 (7.876) | 22M | bilinear |
| SE-ResNet-18 | 71.742 (28.258) | 90.334 (9.666) | 11.8M | bicubic |
| Single-Path NASNet 1.00 | 74.084 (25.916)  | 91.818 (8.182) | 4.42M | bilinear |

### Ported Weights
|Model | Prec@1 (Err) | Prec@5 (Err) | Param # | Image Scaling  | Source |
|---|---|---|---|---|---|
| MNASNet 1.00 (B1) | 72.398 (27.602) | 90.930 (9.070) |  4.36M | bicubic | [Google TFLite](https://github.com/tensorflow/tpu/tree/master/models/official/mnasnet) |
| SE-MNASNet 1.00 (A1) | 73.086 (26.914) | 91.336 (8.664) | 3.87M  | bicubic | [Google TFLite](https://github.com/tensorflow/tpu/tree/master/models/official/mnasnet) |

NOTE: For some reason I can't hit the stated accuracy with my impl of MNASNet and Google's tflite weights. Using a TF equivalent to 'SAME' padding was important to get > 70%, but something small is still missing. Trying to train my own weights from scratch with these models has so far to leveled off in the same 72-73% range.

## TODO
A number of additions planned in the future for various projects, incl
* Find optimal training hyperparams and create/port pretraiend weights for the generic MobileNet variants
* Do a model performance (speed + accuracy) benchmarking across all models (make runable as script)
* More training experiments
* Make folder/file layout compat with usage as a module
* Add usage examples to comments, good hyper params for training
* Comments, cleanup and the usual things that get pushed back
