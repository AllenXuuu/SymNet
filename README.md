# SymNet
As a part of [HAKE](http://hake-mvig.cn/) project (HAKE-Object).

#### **News**: (2020.6.16) Our larger version [HAKE-Large](https://github.com/DirtyHarryLYL/HAKE#hake-large-for-instance-level-hoi-detection) (>120K images, activity and part state labels) is released!

This is the code accompanying the CVPR2020 paper: **Symmetry and Group in Attribute-Object Compositions**. [[arXiv](https://arxiv.org/abs/2004.00587)]

*[Yong-Lu Li](https://dirtyharrylyl.github.io/), [Yue Xu](https://silicx.github.io/), Xiaohan Mao, [Cewu Lu](http://mvig.sjtu.edu.cn/)*

![Overview](./data/overview.png)

If you find this repository useful for you, please consider citing our paper.

	@inproceedings{li2020symmetry,
		title={Symmetry and Group in Attribute-Object Compositions},
		author={Li, Yong-Lu and Xu, Yue and Mao, Xiaohan and Lu, Cewu},
		booktitle={CVPR},
		year={2020}
	}


## Prerequisites

**Packages**: Install using `pip install -r requirements.txt`

**Datasets**: Download and re-arrange with:
	
	cd data; bash download_data.sh

**Features and pretrained models**: Features for compositional ZSL (CZSL) setting<sup>[1]</sup> will be downloaded together with the datasets. Features for generalized compositional ZSL (GCZSL) setting<sup>[2]</sup> can be extracted using:

	python utils/dataset/GCZSL_dataset.py [MIT/UT]

Pretrained models and intermediate results can be downloaded from here: [obj_scores](https://drive.google.com/open?id=1hNcYA9MBfrXXrcf8SlmRlRt4I638bF7H) and [weights](https://drive.google.com/open?id=1Z2DvfPNZTzfZrvzVrqh2jkkLOYl-muT4). Please unzip the `obj_scores.zip` to `./data/obj_scores` and `weights.zip` to `./weights`.


## Compositional Zero-shot Leaning (CZSL)

These are commands for the split and evaluation metrics introduced by [1].

### Training a object classifier

Before training a SymNet model, train an object classifier by running:

	python run_symnet.py --network fc_obj --name MIT_obj_lr3e-3 --data MIT --epoch 1500 --batchnorm --lr 3e-3
	python run_symnet.py --network fc_obj --name UT_obj_lr1e-3 --data UT --epoch 300 --batchnorm --lr 1e-3

Then store the intermediate object results:

	python test_obj.py --network fc_obj --name MIT_obj_lr3e-3 --data MIT --epoch 1120 --batchnorm
	python test_obj.py --network fc_obj --name UT_obj_lr1e-3 --data UT --epoch 140 --batchnorm

The results file will be stored in `./data/obj_scores` with names `MIT_obj_lr3e-3_ep1120.pkl` and `UT_obj_lr1e-3_ep140.pkl` (in the examples above).

### Training a SymNet

To train a SymNet with the hyper-parameters in our paper, run:

	python run_symnet.py --name MIT_best --data MIT --epoch 400 --obj_pred MIT_lr3e-3_ep1120.pkl --batchnorm --lr 5e-4 --bz 512 --lambda_cls_attr 1 --lambda_cls_obj 0.01 --lambda_trip 0.03 --lambda_sym 0.05 --lambda_axiom 0.01
	python run_symnet.py --name UT_best --data UT --epoch 700 --obj_pred UT_lr1e-3_ep140.pkl --batchnorm  --wordvec onehot  --lr 1e-4 --bz 256 --lambda_cls_attr 1 --lambda_cls_obj 0.5 --lambda_trip 0.5 --lambda_sym 0.01 --lambda_axiom 0.03



### Model Evaluation

	python test_symnet.py --name MIT_best --data MIT --epoch 320 --obj_pred MIT_lr3e-3_ep1120.pkl --batchnorm
	python test_symnet.py --name UT_best --data UT --epoch 600 --obj_pred UT_lr1e-3_ep140.pkl --wordvec onehot --batchnorm



Method | MIT (top-1) | MIT (top-2) |MIT (top-2) | UT (top-1) | UT (top-2) | UT (top-3)  
-- | -- | -- | -- | -- | -- | -- |
Visual Product  | 9.8/13.9 | 16.1 | 20.6 | 49.9 | / | / 
LabelEmbed (LE) | 11.2/13.4| 17.6 | 22.4 | 25.8 | / | / 
~- LEOR            | 4.5          | 6.2  | 11.8 |  /       | / | / 
~- LE + R          | 9.3          | 16.3 | 20.8 |  /       | / | / 
~- LabelEmbed+    | 14.8*         |  /   |  /   | 37.4| / | / 
AnalogousAttr | 1.4          |  /   |  /   | 18.3  |  /  |  /  
Red Wine        | 13.1         | 21.2 | 27.6 | 40.3  |  /  |  /   
AttOperator    | 14.2         | 19.6 | 25.1 | 46.2  | 56.6 | 69.2 
TAFE-Net           | 16.4         | 26.4 | 33.0 | 33.2  |  /  |  /  
GenModel       | 17.8         |  /   |  /   | 48.3  |  /  |  /  
**SymNet (Ours)** | **19.9** | **28.2** | **33.8** | **52.1**  |**67.8** |  **76.0** 



## Generalized Compositional Zero-shot Leaning (GCZSL)

These are commands for the split and evaluation metrics introduced by [2].

### Training a object classifier

	python run_symnet.py --network fc_obj --data MITg --name MITg_obj --bz 2048 --lr 3e-3 --epoch 1000 --dropout 0.7  --fc_cls 1024

To store the object classification results of both valid and test set, run:

	python test_obj.py --network fc_obj --data MITg --name MITg_obj --epoch 605 --dropout 0.7 --fc_cls 1024 --test_set test
	python test_obj.py --network fc_obj --data MITg --name MITg_obj --epoch 605 --dropout 0.7 --fc_cls 1024 --test_set val


### Trainig a SymNet
To train a SymNet with the hyper-parameters in our paper, run:

	python run_symnet_gczsl.py --data MITg --name MITg_best --epoch 400 --obj_pred MITg_obj_val_ep605.pkl --test_set val --rmd_metric rmd --lr 5e-4 --bz 512 --batchnorm  --lambda_cls_attr 1 --lambda_cls_obj 0.01 --lambda_trip 0.03 --lambda_sym 0.05 --lambda_axiom 0.01 --test_freq 5


### Model Evaluation

	python test_symnet_gczsl.py --data MITg --name MITg_best --epoch 380 --obj_pred MITg_obj_val_ep605.pkl --batchnorm --test_set val --topk 1
	python test_symnet_gczsl.py --data MITg --name MITg_best --epoch 380 --obj_pred MITg_obj_test_ep605.pkl --batchnorm --test_set test --topk 1


Model | Val Top-1 AUC | Val Top-2 AUC | Val Top-3 AUC | Test Top-1 AUC | Test Top-2 AUC | Test Top-3 AUC | Seen | Unseen | HM
-- | -- | -- | -- | -- | -- | -- | -- | -- | --
AttOperator  | 2.5 | 6.2 | 10.1 | 1.6 | 4.7 | 7.6 | 14.3    | 17.4 | 9.9 
Red Wine      | 2.9 | 7.3 | 11.8 | 2.4 | 5.7 | 9.3 | 20.7    | 17.9 | 11.6
LabelEmbed+  | 3.0 | 7.6 | 12.2 | 2.0 | 5.6 | 9.4 | 15.0    | 20.1 | 10.7
GenModel     | 3.1 | 6.9 | 10.5 | 2.3 | 5.7 | 8.8 | **24.8**    | 13.4 | 11.2
TMN               | 3.5 | 8.1 | 12.4 | 2.9 | 7.1 | 11.5| 20.2    | 20.1 | 13.0
**SymNet (Ours)** | **4.3** | **9.8** | **14.8** | **3.0** | **7.6** | **12.3** | 24.4 | **25.2** | **16.1**


## Acknowledgement
The dataloader and evaluation code are based on [Attributes as Operators](https://github.com/Tushar-N/attributes-as-operators)<sup>[1]</sup> and [Task-Driven Modular Networks](https://github.com/facebookresearch/taskmodularnets)<sup>[2]</sup>.



## Reference

[1] [Attributes as Operators: Factorizing Unseen Attribute-Object Compositions](https://arxiv.org/abs/1803.09851)

[2] [Task-Driven Modular Networks for Zero-Shot Compositional Learning](https://arxiv.org/abs/1905.05908)
