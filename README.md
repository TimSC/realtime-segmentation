# Introduction
PyTorch Implementation of realtime semantic segmentation models, support multi-gpu training and validating, automatic mixed precision training, knowledge distillation etc.  
\
<img src="demo/enet_800epoch.gif" width="100%" height="100%" />


# Requirements
torch == 1.8.1  
segmentation-models-pytorch  
torchmetrics  
albumentations  
loguru  
tqdm  


# Supported models
- [BiSeNetv2](models/bisenetv2.py) [^bisenetv2]  
- [ContextNet](models/contextnet.py)[^contextnet]  
- [DDRNet](models/ddrnet.py)[^ddrnet]  
- [ENet](models/enet.py) [^enet]  
- [FastSCNN](models/fastscnn.py) [^fastscnn]  
- [LEDNet](models/lednet.py) [^lednet]  
- [LinkNet](models/linknet.py)[^linknet]  
- [PP-LiteSeg](models/pp_liteseg.py)[^ppliteseg]  

If you want to use encoder-decoder structure with pretrained encoders, you may refer to: segmentation-models-pytorch[^smp]. This repo also provides easy access to SMP. Just modify the [config file](configs/my_config.py) to (e.g. if you want to train DeepLabv3Plus with ResNet-101 backbone as teacher model to perform knowledge distillation)  
```
self.model = 'smp'
self.encoder = 'resnet101'
self.decoder = 'deeplabv3p'
```


[^bisenetv2]: [BiSeNet V2: Bilateral Network with Guided Aggregation for Real-time Semantic Segmentation](https://arxiv.org/abs/2004.02147)  
[^contextnet]: [ContextNet: Exploring Context and Detail for Semantic Segmentation in Real-time](https://arxiv.org/abs/1805.04554)  
[^ddrnet]: [Deep Dual-resolution Networks for Real-time and Accurate Semantic Segmentation of Road Scenes](https://arxiv.org/abs/2101.06085)  
[^enet]: [ENet: A Deep Neural Network Architecture for Real-Time Semantic Segmentation](https://arxiv.org/abs/1606.02147)  
[^fastscnn]: [Fast-SCNN: Fast Semantic Segmentation Network](https://arxiv.org/abs/1902.04502)  
[^lednet]: [LEDNet: A Lightweight Encoder-Decoder Network for Real-Time Semantic Segmentation](https://arxiv.org/abs/1905.02423)  
[^linknet]: [LinkNet: Exploiting Encoder Representations for Efficient Semantic Segmentation](https://arxiv.org/abs/1707.03718)  
[^ppliteseg]: [PP-LiteSeg: A Superior Real-Time Semantic Segmentation Model](https://arxiv.org/abs/2204.02681)  
[^smp]: [segmentation-models-pytorch](https://github.com/qubvel/segmentation_models.pytorch)  


# Knowledge Distillation
Currently only support the original knowledge distillation method proposed by Geoffrey Hinton.[^kd]  

[^kd]: [Distilling the Knowledge in a Neural Network](https://arxiv.org/abs/1503.02531)  


# How to use
## DDP training (recommend)
```
CUDA_VISIBLE_DEVICES=0,1,2,3 python -m torch.distributed.launch --nproc_per_node=4 main.py
```

## DP training
```
CUDA_VISIBLE_DEVICES=0,1,2,3 python main.py
```


# Performances and checkpoints  
## full resolution on Cityscapes  
| Model | Params (M) <br> my / paper | mIoU (paper) <br> val / test| mIoU (200 epoch) <br> crop 768x768 | mIoU (800 epoch) <br> crop 768x768 | mIoU (800 epoch) <br> crop 1024x1024 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| BiSeNetv2 | 2.53 / N/A | 73.4 / 72.6 | 69.36* | [72.15*](weights/bisenetv2-aux_800epoch.pth) | [73.73*](weights/bisenetv2-aux_crop-1024_800epoch.pth) |
| ContextNet | 1.01 / 0.85 | 65.9 / 66.1 | 62.17 | [66.15](weights/contextnet_800epoch.pth) | [66.61](weights/contextnet_crop-1024_800epoch.pth) |
| DDRNet | 5.54 / 5.7 | 77.8 / 77.4 | 68.40* | [73.70*](weights/ddrnet-23-slim_800epoch.pth) | [74.34](weights/ddrnet-23-slim_crop-1024_800epoch.pth) |
| ENet | 0.37 / 0.37 | N/A / 58.3 | 62.03 | [69.65](weights/enet_800epoch.pth) | [71.31](weights/enet_crop-1024_800epoch.pth) |
| FastSCNN | 1.02 / 1.11 | 68.6 / 68.0 | 61.31 | [66.75](weights/fastscnn_800epoch.pth) | [69.37](weights/fastscnn_crop-1024_800epoch.pth) |
| LEDNet | 1.46 / 0.94 | N/A / 70.6 | 65.91 | [71.76](weights/lednet_800epoch.pth) | [71.40](weights/lednet_crop-1024_800epoch.pth) |
| LinkNet | 11.71 / 11.5 | N/A / 76.4| 63.82 | [70.86](weights/linknet_800epoch.pth) | [71.72](weights/linknet_crop-1024_800epoch.pth) |
| PP-LiteSeg <br> Enc: STDC1 | 6.33 / N/A | 76.0** / 74.9**| 62.91 | [69.07](weights/ppliteseg_stdc1_800epoch.pth) | [72.49](weights/ppliteseg_stdc1_crop-1024_800epoch.pth) |
| PP-LiteSeg <br> Enc: STDC2 | 10.56 / N/A | 78.2** / 77.5**| 63.36 | [72.42](weights/ppliteseg_stdc2_800epoch.pth) | [74.37](weights/ppliteseg_stdc2_crop-1024_800epoch.pth) |

[*These results are obtained by using auxiliary heads]  
[**These results are obtained at resolution 768 × 1536]  


## SMP performance on Cityscapes  
| Decoder | Params (M) | mIoU (200 epoch) | mIoU (800 epoch) |
| :---: | :---: | :---: | :---: |
| DeepLabv3 | 15.90 | 75.22 | 77.16 |
| DeepLabv3Plus | 12.33 | 73.97 | 75.90 |
| FPN | 13.05 | 73.44 | 74.94 |
| LinkNet | 11.66 | 71.17 | 73.19 |
| MANet | 21.68 | 74.59 | 76.14 |
| PAN | 11.37 | 70.25 | 72.46 |
| PSPNet | 11.41 | 61.63 | 67.26 |
| UNet | 14.33 | 72.99 | 74.45 |
| UNetPlusPlus | 15.97 | 74.31 | 75.57 |

[For comparison, the above results are all using ResNet-18 as encoders.]  


## Knowledge distillation  
| Model | Encoder | Decoder | kd_training | mIoU (200 epoch) | mIoU (800 epoch) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| SMP | DeepLabv3Plus | ResNet-101 <br> teacher | - | 78.10 | 79.20 |
| SMP | DeepLabv3Plus | ResNet-18 <br> student | False | 73.97 | 75.90 |
| SMP | DeepLabv3Plus | ResNet-18 <br> student | True | 75.20 | 76.41 |


# Prepare the dataset
```
/Cityscapes
    /gtFine
    /leftImg8bit
```



# References