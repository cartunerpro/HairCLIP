# HairCLIP: Design Your Hair by Text and Reference Image (CVPR2022)
<a href="https://openaccess.thecvf.com/content/CVPR2022/papers/Wei_HairCLIP_Design_Your_Hair_by_Text_and_Reference_Image_CVPR_2022_paper.pdf"><img src="https://img.shields.io/badge/Paper-CVPR2022-blue.svg"></a>
> This repository hosts the official PyTorch implementation of the paper: "**HairCLIP: Design Your Hair by Text and Reference Image**".

Our **single** framework supports **hairstyle and hair color editing** individually or jointly, and conditional inputs can come from either **image** or **text** domain. 

<img src='assets/teaser.png'>


Tianyi Wei<sup>1</sup>,
Dongdong Chen<sup>2</sup>,
Wenbo Zhou<sup>1</sup>,
Jing Liao<sup>3</sup>,
Zhentao Tan<sup>1</sup>,
Lu Yuan<sup>2</sup>, 
Weiming Zhang<sup>1</sup>, 
Nenghai Yu<sup>1</sup> <br>
<sup>1</sup>University of Science and Technology of China, <sup>2</sup>Microsoft Cloud AI, <sup>3</sup>City University of Hong Kong

## News
**`2023.10.12`**: We propose the more performant [HairCLIPv2](https://github.com/wty-ustc/HairCLIPv2) that supports various interaction modalities, which is accepted by ICCV2023! 🎉  
**`2022.03.19`**: Our testing code and pretrained model are released.  
**`2022.03.09`**: Our training code is released.  
**`2022.03.02`**: Our paper is accepted by CVPR2022 and the code will be released soon.   

## Web Demo
Upload your own image and try HairCLIP here with Replicate [![Replicate](https://replicate.com/wty-ustc/hairclip/badge)](https://replicate.com/wty-ustc/hairclip)

## Getting Started
### Prerequisites
```bash
$ conda install --yes -c pytorch pytorch=1.7.1 torchvision cudatoolkit=11.0
$ pip install ftfy regex tqdm
$ pip install git+https://github.com/openai/CLIP.git
$ pip install tensorflow-io
```
### Pretrained Model
Please download the pre-trained model from the following link. The HairCLIP model contains the entire architecture, including the mapper and decoder weights.
| Path | Description
| :--- | :----------
|[HairCLIP](https://drive.google.com/file/d/1hqZT6ZMldhX3M_x378Sm4Z2HMYr-UwQ4/view?usp=sharing)  | Our pre-trained HairCLIP model.  

If you wish to use the pretrained model for training or inference, you may do so using the flag `--checkpoint_path`.  
### Auxiliary Models and Latent Codes
In addition, we provide various auxiliary models and latent codes inverted by [e4e](https://github.com/omertov/encoder4editing) needed for training your own HairCLIP model from scratch.
| Path | Description
| :--- | :----------
|[FFHQ StyleGAN](https://drive.google.com/file/d/1pts5tkfAcWrg4TpLDu6ILF5wHID32Nzm/view?usp=sharing) | StyleGAN model pretrained on FFHQ taken from [rosinality](https://github.com/rosinality/stylegan2-pytorch) with 1024x1024 output resolution.
|[IR-SE50 Model](https://drive.google.com/file/d/1FS2V756j-4kWduGxfir55cMni5mZvBTv/view?usp=sharing) | Pretrained IR-SE50 model taken from [TreB1eN](https://github.com/TreB1eN/InsightFace_Pytorch) for use in our ID loss during HairCLIP training.
|[Train Set](https://drive.google.com/file/d/1gof8kYc_gDLUT4wQlmUdAtPnQIlCO26q/view?usp=sharing) | CelebA-HQ train set latent codes inverted by [e4e](https://github.com/omertov/encoder4editing).
|[Test Set](https://drive.google.com/file/d/1j7RIfmrCoisxx3t-r-KC02Qc8barBecr/view?usp=sharing) | CelebA-HQ test set latent codes inverted by [e4e](https://github.com/omertov/encoder4editing).  

By default, we assume that all auxiliary models are downloaded and saved to the directory `pretrained_models`.
## Training
### Training HairCLIP
The main training script can be found in `scripts/train.py`.   
Intermediate training results are saved to `opts.exp_dir`. This includes checkpoints, train outputs, and test outputs.  
Additionally, if you have tensorboard installed, you can visualize tensorboard logs in `opts.exp_dir/logs`.
#### **Training the HairCLIP Mapper**
```bash
cd mapper
python scripts/train.py \
--exp_dir=/path/to/experiment \
--hairstyle_description="hairstyle_list.txt" \
--color_description="purple, red, orange, yellow, green, blue, gray, brown, black, white, blond, pink" \
--latents_train_path=/path/to/train_faces.pt \
--latents_test_path=/path/to/test_faces.pt \
--hairstyle_ref_img_train_path=/path/to/celeba_hq_train \
--hairstyle_ref_img_test_path=/path/to/celeba_hq_val \
--color_ref_img_train_path=/path/to/celeba_hq_train \
--color_ref_img_test_path=/path/to/celeba_hq_val \
--color_ref_img_in_domain_path=/path/to/generated_hair_of_various colors \
--hairstyle_manipulation_prob=0.5 \
--color_manipulation_prob=0.2 \
--both_manipulation_prob=0.27 \
--hairstyle_text_manipulation_prob=0.5 \
--color_text_manipulation_prob=0.5 \
--color_in_domain_ref_manipulation_prob=0.25 \
```
### Additional Notes
- **This version only supports batch size and test batch size to be 1.**
- See `options/train_options.py` for all training-specific flags. 
- See `options/test_options.py` for all test-specific flags.
- You can customize your own HairCLIP by adjusting the different category probabilities. For example, if you want to train a HairCLIP that only performs hair color editing with text as the interaction mode, you can adjust the different probabilities as follows.
  ```
  --hairstyle_manipulation_prob=0 \
  --color_manipulation_prob=1 \
  --both_manipulation_prob=0 \
  --color_text_manipulation_prob=1 \
  ```
- `--color_ref_img_in_domain_path` is a dataset of images with diverse hair colors generated by the HairCLIP trained from the above probabilistic configuration to enhance the diversity of the dataset when editing hair colors based on the reference image, which you may not use. If you choose to use this augmentation, you need to pre-train a text-based hair color HairCLIP according to above probabilistic configuration.
- The weights of different losses in training are in the `options/train_options.py`, and you can adjust them to balance your needs. Empirically, the larger the loss weight, the better the corresponding effect, but it will affect the effect of other losses to some extent.
## Testing
### Inference
The main inference script can be found in `scripts/inference.py`. Inference results are saved to `test_opts.exp_dir`.  
#### Example of Using Text to Edit Hairstyle
```bash
cd mapper
python scripts/inference.py \
--exp_dir=/path/to/experiment \
--checkpoint_path=../pretrained_models/hairclip.pt \
--latents_test_path=/path/to/test_faces.pt \
--editing_type=hairstyle \
--input_type=text \
--hairstyle_description="hairstyle_list.txt" \
```
#### Example of Using Text to Edit Hairstyle Reference Image to Edit Hair Color
```bash
cd mapper
python scripts/inference.py \
--exp_dir=/path/to/experiment \
--checkpoint_path=../pretrained_models/hairclip.pt \
--latents_test_path=/path/to/test_faces.pt \
--editing_type=both \
--input_type=text_image \
--hairstyle_description="hairstyle_list.txt" \
--color_ref_img_test_path=/path/to/celeba_hq_test \
```
### Additional Notes
- See `options/test_options.py` for all test-specific flags.
- `--editing_type` should be `hairstyle`, `color`, or `both` to indicate whether to edit only hairstyle, only hair color, or both hairstyle and hair color.
- `--input_type` is used to indicate the interaction mode, `text` for text, and `image` for reference image. When editing both hairstyle and hair color, the two interactions are separated by `_`.
- The `--start_index` and `--end_index` indicate the range of the edited test latent codes, where `--start_index` needs to be greater than 0 and `--end_index` cannot exceed the size of the whole test latent codes dataset.

## Acknowledgements
This code is based on [StyleCLIP](https://github.com/orpatashnik/StyleCLIP).

## Citation

If you find our work useful for your research, please consider citing the following papers :)

```
@article{wei2022hairclip,
  title={Hairclip: Design your hair by text and reference image},
  author={Wei, Tianyi and Chen, Dongdong and Zhou, Wenbo and Liao, Jing and Tan, Zhentao and Yuan, Lu and Zhang, Weiming and Yu, Nenghai},
  journal={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  year={2022}
}
```

```
@article{wei2023hairclipv2,
  title={HairCLIPv2: Unifying Hair Editing via Proxy Feature Blending},
  author={Wei, Tianyi and Chen, Dongdong and Zhou, Wenbo and Liao, Jing and Zhang, Weiming and Hua, Gang and Yu, Nenghai},
  journal={Proceedings of the IEEE/CVF International Conference on Computer Vision},
  year={2023}
}
```
