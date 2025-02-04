
## Using LoRA to fine-tune HunyuanDiT


### Instructions

 The dependencies and installation are basically the same as the [**original model**](https://huggingface.co/Tencent-Hunyuan/HunyuanDiT-v1.1).

 We provide two types of trained LoRA weights for you to test.
 
 Then download the model using the following commands:

```bash
cd HunyuanDiT
# Use the huggingface-cli tool to download the model.
huggingface-cli download Tencent-Hunyuan/HYDiT-LoRA --local-dir ./ckpts/t2i/lora

# Quick start
python sample_t2i.py --prompt "青花瓷风格，一只猫在追蝴蝶"  --no-enhance --load-key ema --lora_ckpt ./ckpts/t2i/lora/porcelain
```

Examples of training data and inference results are as follows:
<table>
  <tr>
    <td colspan="4" align="center">Examples of training data</td>
  </tr>
  
  <tr>
    <td align="center"><img src="asset/porcelain/train/0.png" alt="Image 0" width="200"/></td>
    <td align="center"><img src="asset/porcelain/train/1.png" alt="Image 1" width="200"/></td>
    <td align="center"><img src="asset/porcelain/train/2.png" alt="Image 2" width="200"/></td>
    <td align="center"><img src="asset/porcelain/train/3.png" alt="Image 3" width="200"/></td>
  </tr>
  <tr>
    <td align="center">青花瓷风格，一只蓝色的鸟儿站在蓝色的花瓶上，周围点缀着白色花朵，背景是白色 </td>
    <td align="center">青花瓷风格，这是一幅蓝白相间的陶瓷盘子，上面描绘着一只狐狸和它的幼崽在森林中漫步，背景是白色 </td>
    <td align="center">青花瓷风格，在黑色背景上，一只蓝色的狼站在蓝白相间的盘子上，周围是树木和月亮 </td>
    <td align="center">青花瓷风格，在蓝色背景上，一只蓝色蝴蝶和白色花朵被放置在中央 </td>
  </tr>
  <tr>
    <td colspan="4" align="center">Examples of inference results</td>
  </tr>
  <tr>
    <td align="center"><img src="asset/porcelain/inference/0.png" alt="Image 4" width="200"/></td>
    <td align="center"><img src="asset/porcelain/inference/1.png" alt="Image 5" width="200"/></td>
    <td align="center"><img src="asset/porcelain/inference/2.png" alt="Image 6" width="200"/></td>
    <td align="center"><img src="asset/porcelain/inference/3.png" alt="Image 7" width="200"/></td>
  </tr>
  <tr>
    <td align="center">青花瓷风格，苏州园林</td>
    <td align="center">青花瓷风格，一朵荷花</td>
    <td align="center">青花瓷风格，一只羊</td>
    <td align="center">青花瓷风格，一个女孩在雨中跳舞</td>
  </tr>
  
</table>


### Training
    
We provide three types of weights for fine-tuning LoRA, `ema`, `module` and `distill`, and you can choose according to the actual effect. By default, we use `ema` weights. 

Here is an example, we load the `ema` weights into the main model and perform LoRA fine-tuning through the `--ema-to-module` parameter. 

If you want to load the `module` weights into the main model, just remove the `--ema-to-module` parameter.

If multiple resolution are used, you need to add the `--multireso` and `--reso-step 64 ` parameter. 

```bash
model='DiT-g/2'                                        # model type
task_flag="lora_porcelain_ema_rank64"                  # task flag 
resume=./ckpts/t2i/model/                              # resume checkpoint 
index_file=dataset/porcelain/jsons/porcelain.json      # the selected data indices
results_dir=./log_EXP                                  # save root for results
batch_size=1                                           # training batch size
image_size=1024                                        # training image resolution
grad_accu_steps=2                                      # gradient accumulation steps
warmup_num_steps=0                                     # warm-up steps
lr=0.0001                                              # learning rate
ckpt_every=100                                         # create a ckpt every a few steps.
ckpt_latest_every=2000                                 # create a ckpt named `latest.pt` every a few steps.
rank=64                                                # rank of lora
max_training_steps=2000                                # Maximum training iteration steps

PYTHONPATH=./ deepspeed hydit/train_deepspeed.py \
    --task-flag ${task_flag} \
    --model ${model} \
    --training_parts lora \
    --rank ${rank} \
    --resume-split \
    --resume ${resume} \
    --ema-to-module \
    --lr ${lr} \
    --noise-schedule scaled_linear --beta-start 0.00085 --beta-end 0.03 \
    --predict-type v_prediction \
    --uncond-p 0.44 \
    --uncond-p-t5 0.44 \
    --index-file ${index_file} \
    --random-flip \
    --batch-size ${batch_size} \
    --image-size ${image_size} \
    --global-seed 999 \
    --grad-accu-steps ${grad_accu_steps} \
    --warmup-num-steps ${warmup_num_steps} \
    --use-flash-attn \
    --use-fp16 \
    --ema-dtype fp32 \
    --results-dir ${results_dir} \
    --ckpt-every ${ckpt_every} \
    --max-training-steps ${max_training_steps}\
    --ckpt-latest-every ${ckpt_latest_every} \
    --log-every 10 \
    --deepspeed \
    --deepspeed-optimizer \
    --use-zero-stage 2 \
    --qk-norm \
    --rope-img base512 \
    --rope-real \
    "$@"
```

Recommended parameter settings

|     Parameter     |  Description  |          Recommended Parameter Value                               | Note|
|:---------------:|:---------:|:---------------------------------------------------:|:--:|
|   `--batch_size` |    Training batch size    |        1        | Depends on GPU memory|
|   `--grad-accu-steps` |    Size of gradient accumulation    |       2        | - |
|   `--rank` |    Rank of lora    |       64        | Choosing from 8-128 |
|   `--max-training-steps` |    Training steps  |       2000        | Depend on training data size, for reference apply 2000 steps on 100 images|
|   `--lr` |    Learning rate  |        0.0001        | - |


### Inference

After the training is complete, you can use the following command line for inference.
We provide the `--lora_ckpt` parameter for selecting the folder which contains lora weights and configurations.

a. Using LoRA during inference

```bash
python sample_t2i.py --prompt "青花瓷风格，一只小狗"  --no-enhance --lora_ckpt log_EXP/001-lora_porcelain_ema_rank64/checkpoints/0001000.pt
```

b. Using LoRA in gradio
```bash
python app/hydit_app.py --infer-mode fa  --no-enhance --lora_ckpt log_EXP/001-lora_porcelain_ema_rank64/checkpoints/0001000.pt
```

c. Merge LoRA weights into the main model

We provide the `--output_merge_path` parameter to set the path for saving the merged weights.

```bash
PYTHONPATH=./ python lora/merge.py --lora_ckpt log_EXP/001-lora_porcelain_ema_rank64/checkpoints/0001000.pt --output_merge_path ./ckpts/t2i/model/pytorch_model_merge.pt
```

d. For more information, please refer to [HYDiT-LoRA](https://huggingface.co/Tencent-Hunyuan/HYDiT-LoRA).
