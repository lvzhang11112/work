# ST-Pruning:Representation-Aware Timestep Pruning for Efficient Diffusion Models
<div align="center">
<img src="assets/framework.png" width="80%"></img>
</div>
This work presents *ST-Pruning*, an efficient structrual pruning method for diffusion models. Our empirical assessment highlights two primary features:
1) ``Efficiency``: It enables approximately a 50% reduction in FLOPs at a mere 10% to 20% of the original training expenditure; 
2) ``Consistency``: The pruned diffusion models inherently preserve generative behavior congruent with the pre-trained ones.

<div align="center">
<img src="assets/LSUN.png" width="80%"></img>
</div>

### Supported Methods
- [x] Magnitude Pruning
- [x] Random Pruning
- [x] Taylor Pruning
- [x] Diff-Pruning   
- [x] ST-Pruning (A taylor-based method proposed in our paper)  
### TODO List
- [ ] Support more diffusion models from Diffusers
- [ ] Training scripts for  LSUN Church & LSUN Bedroom
- [ ] Align the performance with the [DDIM Repo](https://github.com/ermongroup/ddim). 

## Pruning with Huggingface Diffusers

The following pipeline prunes a pre-trained DDPM on CIFAR-10 with [Huggingface Diffusers](https://github.com/huggingface/diffusers).

### 0. Environment, Data and Pretrained Model
* Requirements
```bash
conda env create -f environment.yaml
```
* Data
  
Download and extract CIFAR-10 images to *data/cifar10_images* for training and evaluation.
```bash
python tools/extract_cifar10.py --output data
```
* Pretrained Models
   You can also download a pre-converted model using wget
```bash
wget https://github.com/lvzhang11112/Slide-pruning/releases/download/Modles/ddpm_ema_cifar10.zip
wget https://github.com/lvzhang11112/Slide-pruning/releases/download/Modles/ddpm-ema-bedroom.zip
wget https://github.com/lvzhang11112/Slide-pruning/releases/download/Modles/ddpm-ema-church.zip
```
* Pruned Models
   You can also download a pruned model using wget
```bash
wget https://github.com/lvzhang11112/Slide-pruning/releases/download/Modles/ddpm_cifar10_pruned.zip
wget https://github.com/lvzhang11112/Slide-pruning/releases/download/Modles/ddpm_ema_bedroom_256_pruned.zip
wget https://github.com/lvzhang11112/Slide-pruning/releases/download/Modles/ddpm_ema_church_256_pruned.zip
```
### 1. Pruning
Create a pruned model at *run/pruned/ddpm_cifar10_pruned*
```bash
bash scripts/prune_ddpm_cifar10.sh 0.3  # pruning ratio = 30\%
```

### 2. Finetuning (Post-Training)
Finetune the model and save it at *run/finetuned/ddpm_cifar10_pruned_post_training*
```bash
bash scripts/finetune_ddpm_cifar10.sh
```

### 3. Sampling
**Pruned:** Sample and save images to *run/sample/ddpm_cifar10_pruned*
```bash
bash scripts/sample_ddpm_cifar10_pruned.sh
```

**Pretrained:** Sample and save images to *run/sample/ddpm_cifar10_pretrained*
```bash
bash scripts/sample_ddpm_cifar10_pretrained.sh
```

### 4. FID Score
This script was modified from https://github.com/mseitzer/pytorch-fid. 

```bash
# pre-compute the stats of CIFAR-10 dataset
python fid_score.py --save-stats data/cifar10_images run/fid_stats_cifar10.npz --device cuda:0 --batch-size 256
```

```bash
# Compute the FID score of sampled images
python fid_score.py run/sample/ddpm_cifar10_pruned run/fid_stats_cifar10.npz --device cuda:0 --batch-size 256
```

### 5. (Optional) Distributed Training and Sampling with Accelerate
This project supports distributed training and sampling. 
```bash
python -m torch.distributed.launch --nproc_per_node=8 --master_port 22222 --use_env <ddpm_sample.py|ddpm_train.py> ...
```
A multi-processing example can be found at [scripts/sample_ddpm_cifar10_pretrained_distributed.sh](scripts/sample_ddpm_cifar10_pretrained_distributed.sh).


## Prune Pre-trained DPMs from [HuggingFace Diffusers](https://huggingface.co/models?library=diffusers)

### :rocket: [Denoising Diffusion Probabilistic Models (DDPMs)](https://arxiv.org/abs/2006.11239)
Example: [google/ddpm-ema-bedroom-256](https://huggingface.co/google/ddpm-ema-bedroom-256)
```bash
python ddpm_prune.py \
--dataset "<path/to/imagefoler>" \  
--model_path google/ddpm-ema-bedroom-256 \
--save_path run/pruned/ddpm_ema_bedroom_256_pruned \
--pruning_ratio 0.05 \
--pruner "<random|magnitude|reinit|taylor|slide-pruning|diff-pruning>" \
--batch_size 4 \
--thr 0.05 \
--device cuda:0 \
```
The ``dataset`` and ``thr`` arguments only work for taylor & diff-pruning & slide-pruning.

## Acknowledgement

This project is heavily based on [Diffusers](https://github.com/huggingface/diffusers), [Torch-Pruning](https://github.com/VainF/Torch-Pruning), [pytorch-fid](https://github.com/mseitzer/pytorch-fid). Our experiments were conducted on [ddim](https://github.com/ermongroup/ddim) and [LDM](https://github.com/CompVis/latent-diffusion).


