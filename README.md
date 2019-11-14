# Mask RCNN 

Performance focused implementation of Mask RCNN based on the [Tensorpack implementation](https://github.com/tensorpack/tensorpack/tree/master/examples/FasterRCNN).
The original paper: [Mask R-CNN](https://arxiv.org/abs/1703.06870)
### Overview

This implementation of Mask RCNN is focused on increasing training throughput without sacrificing any accuracy. We do this by training with a batch size > 1 per GPU using FP16 and two custom TF ops.

### Status

Training on N GPUs (V100s in our experiments) with a per-gpu batch size of M = NxM training

Training converges to target accuracy for configurations from 8x1 up to 32x4 training. Training throughput is substantially improved from original Tensorpack code.

A pre-built dockerfile is available in DockerHub under `awssamples/mask-rcnn-tensorflow:latest`. It is automatically built on each commit to master.

### Notes

- Running this codebase requires a custom TF binary - available under GitHub releases
  - The custom_op.patch contains the git diff from our custom TF
  - There are also pre-built TF wheels, the stable version is on TF 1.14.
- We give some details the codebase and optimizations in `CODEBASE.md`

### To launch training
- Data preprocessing
  - We are using COCO 2017, you can download the data from [COCO data](http://cocodataset.org/#download).
  - The pre-trained resnet backbone can be downloaded from [ImageNet-R50-AlignPadding.npz](http://models.tensorpack.com/FasterRCNN/ImageNet-R50-AlignPadding.npz)
  - The file folder needs to have the following directory structure:
  ```
  data/
    annotations/
      instances_train2017.json
      instances_val2017.json
    pretrained-models/
      ImageNet-R50-AlignPadding.npz
    train2017/
      # image files that are mentioned in the corresponding json
    val2017/
      # image files that are mentioned in corresponding json
  ```
  - If you want to use COCO 2014, please refer to [here](https://github.com/tensorpack/tensorpack/tree/master/examples/FasterRCNN)
  - If you want to use EKS or Sagemaker, you need to create your own S3 bucket which contains the data in the same directory structure, and change the S3 bucket name in the following files:
    - EKS: [stage-data](https://github.com/aws-samples/mask-rcnn-tensorflow/blob/master/infra/eks/fsx/stage-data.yaml)
    - SageMaker: [S3 download](https://github.com/aws-samples/mask-rcnn-tensorflow/blob/master/infra/sm/run_mpi.py#L122)
  - If you want to use EKS, you also need to create the a FSx filesystem
    - You don't need to link your S3 bucket if you have followed the previous steps
    - You need to change the FSx filesystem id in [pv-fsx](https://github.com/aws-samples/mask-rcnn-tensorflow/blob/master/infra/eks/fsx/pv-fsx.yaml) file.
- Container is highly recommended for training
  - If you want to build your own image, please refer to our [Dockerfile](https://github.com/aws-samples/mask-rcnn-tensorflow/blob/master/Dockerfile). Please note that you need to rebuild Tensorflow when building the docker image, it is time-consuming.
  - Alternatively, you can use our pre-built docker image: `fewu/mask-rcnn-tensorflow:master-latest`.
- To run on AWS
  - To train with docker on EC2 (best performance), refer to [Docker](https://github.com/aws-samples/mask-rcnn-tensorflow/tree/master/infra/docker)
  - To train with Amazon EKS, refer to [EKS](https://github.com/aws-samples/mask-rcnn-tensorflow/tree/master/infra/eks)
  - To train with Amazon SageMaker, refer to [SageMaker](https://github.com/aws-samples/mask-rcnn-tensorflow/tree/master/infra/sm)

### Training results
The result was running on P3dn.24xl instances using EKS.
12 epochs training:

| Num_GPUs x Images_Per_GPU | Training time | Box mAP | Mask mAP |
| ------------- | ------------- | ------------- | ------------- |
| 8x4 | 5.09h | 37.47% | 34.45% |
| 16x4 | 3.11h | 37.41% | 34.47% |
| 32x4 | 1.94h | 37.20% | 34.25% |

24 epochs training:

| Num_GPUs x Images_Per_GPU | Training time | Box mAP | Mask mAP |
| ------------- | ------------- | ------------- | ------------- |
| 8x4 | 9.78h | 38.25% | 35.08% |
| 16x4 | 5.60h | 38.44% | 35.18% |
| 32x4 | 3.33h | 38.33% | 35.12% |

### Tensorpack fork point

Forked from the excellent Tensorpack repo at commit a9dce5b220dca34b15122a9329ba9ff055e8edc6
