## Omni-GAN

### Comparision of several cGANs

![](.github/cgan_loss.jpg)

#### Results on CIFAR100
<p float="left">
<img src="https://github.com/PeterouZh/Omni-GAN-PyTorch/blob/main/.github/GAN_cGAN_cifar100_IS.png" width="350" />
<img src="https://github.com/PeterouZh/Omni-GAN-PyTorch/blob/main/.github/GAN_cGAN_cifar100_FID.png" width="350" />
</p>

#### More results
![](.github/cifar10_cifar100.png)

## Let's go!

### Environments


```bash
cd Omni-GAN-PyTorch
conda create -y --name omnigan python=3.6.7 
conda activate omnigan
pip install torch==1.6.0+cu101 torchvision==0.7.0+cu101 -f https://download.pytorch.org/whl/torch_stable.html
python -m pip install detectron2 -f https://dl.fbaipublicfiles.com/detectron2/wheels/cu101/torch1.6/index.html
pip install -r requirements.txt
```
- cuda 10.0
- cudnn 7.6.5

Cuda and cudnn are needed by the TensorFlow which is only utilized for the sake of evaluation (monitoring the training process). We recommend downloading cuda and cudnn from the official website and installing them manually.

### Prepare FID statistics files

We provide the FID statistics files calculated on CIFAR10 and CIFAR100 at [OneDrive](https://sjtueducn-my.sharepoint.com/:f:/g/personal/zhoupengcv_sjtu_edu_cn/Ek0QSX1UhylDjVYdmXYxRtcBMLs54AYD4E3CwZlWBXZmPA?e=BzWa9D) respectively. Download them and put them into the `datasets` dir. 
Of course, you can calculate these files by yourself. Below is the command. 

- CIFAR100
```bash
export LD_LIBRARY_PATH=$HOME/.keras/envs/cuda-10.0/lib64:$HOME/.keras/envs/cudnn-10.0-linux-x64-v7.6.5.32/lib64
export CUDA_VISIBLE_DEVICES=3
export PYTHONPATH=./
python template_lib/v2/GAN/evaluation/tf_FID_IS_score.py \
  --tl_config_file configs/prepare_files.yaml \
  --tl_command calculate_fid_stat_CIFAR \
  --tl_outdir results/calculate_fid_stat_CIFAR100 \
  --tl_opts dataset_name cifar100_train \
            dataset_mapper_cfg.name CIFAR100DatasetMapper \
            GAN_metric.tf_fid_stat datasets/fid_stats_tf_cifar100_train_32.npz \
            GAN_metric.tf_inception_model_dir datasets/tf_inception_model
```

- CIFAR10
```bash
export LD_LIBRARY_PATH=$HOME/.keras/envs/cuda-10.0/lib64:$HOME/.keras/envs/cudnn-10.0-linux-x64-v7.6.5.32/lib64
export CUDA_VISIBLE_DEVICES=2
export PYTHONPATH=./
python template_lib/v2/GAN/evaluation/tf_FID_IS_score.py \
  --tl_config_file configs/prepare_files.yaml \
  --tl_command calculate_fid_stat_CIFAR \
  --tl_outdir results/calculate_fid_stat_CIFAR10 \
  --tl_opts dataset_name cifar10_train \
            dataset_mapper_cfg.name CIFAR10DatasetMapper \
            GAN_metric.tf_fid_stat datasets/fid_stats_tf_cifar10_train_32.npz \
            GAN_metric.tf_inception_model_dir datasets/tf_inception_model
```


### Train on CIFAR100
```bash
export CUDA_VISIBLE_DEVICES=2
export PYTHONPATH=./:./BigGAN_PyTorch_1_lib 
export LD_LIBRARY_PATH=$HOME/.keras/envs/cuda-10.0/lib64:$HOME/.keras/envs/cudnn-10.0-linux-x64-v7.6.5.32/lib64 
python exp/unified_loss/train.py \
  --tl_config_file configs/omni_gan_cifar100.yaml \
  --tl_command train_Omni_GAN \
  --tl_outdir results/train_Omni_GAN_cifar100 \
  --tl_opts GAN_metric.tf_fid_stat datasets/fid_stats_tf_cifar100_train_32.npz \
            GAN_metric.tf_inception_model_dir datasets/tf_inception_model \
            args.data_root datasets/cifar100

```


### Train on CIFAR10
```bash
export CUDA_VISIBLE_DEVICES=1
export PYTHONPATH=./:./BigGAN_PyTorch_1_lib 
export LD_LIBRARY_PATH=$HOME/.keras/envs/cuda-10.0/lib64:$HOME/.keras/envs/cudnn-10.0-linux-x64-v7.6.5.32/lib64 
python exp/unified_loss/train.py \
  --tl_config_file configs/omni_gan_cifar10.yaml \
  --tl_command train_Omni_GAN \
  --tl_outdir results/train_Omni_GAN_cifar10 \
  --tl_opts GAN_metric.tf_fid_stat datasets/fid_stats_tf_cifar10_train_32.npz \
            GAN_metric.tf_inception_model_dir datasets/tf_inception_model \
            args.data_root datasets/cifar10

```

Notes:

- Please put the ImageNet dataset `ILSVRC/Data/CLS-LOC/train` into `~/biggan/` dir, where `~` means your home dir. You can change this position by modifying `line 7` of `exp/biggan_to_huawei/BigGAN_ImageNet128_to_HuaWei.yaml`.
- After the program finishes, two files will be saved in `~/biggan/`, i.e., `ILSVRC_I128_index.npz` and `ILSVRC_I128.hdf5` (its size is about 63G). 
- This may take a few minutes, please save the files so that you don't have to repeat this procedure every time you train.


