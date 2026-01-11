# 1.TensorFloat-32（TF32）加速
再Ampere架构如A100 ，RTX 30/40系列的gpu上，TF32可以显著加速FP32矩阵的乘法，同时保持足够的精读
```
torch.backends.cuda.matmul.allow_tf32 = True
torch.backends.cudnn.allow_tf32 = True
```
# 2.torch .compile编译优化
这是PyTorch2.0+的杀手级别的特性，给flux的核心组件transform启用了编译模式

mode = “reduce-overhead”这个模式在使用cuda graph减少python和gpu之间的调度开销，overhead非常适合推理循环

效果，首次推理会变慢，进行编译，随后的推理速度预计提升10-20%

```
pipe.transformer = torch.compile(pipe.transformer, mode="reduce-overhead")
```
# 3/VAE Tiling(显存安全网）
flux的vae解码器在处理高分辨率的图像容易导致oom显存溢出

启动了pipe.enable_vae_tiling()这会将图像切块解码，虽然解码步骤稍微变慢一点点，但是大幅降低峰值显存，确保服务不崩溃

# 4.精度和attention优化

精度：强制模型加载为torch.bfloat16，这是目前生成模型的最佳实践，比fp16更稳，比fp32块并且省显存

attention：diffusers库已经默认集成pytorch的scaled_dot_prod_uct_attention(SDPA)不需要额外安装xformers
就可以享受原生加速，

nvidia-smi确认显存占用平稳的抑郁cpu_offloead和vae_tiling

