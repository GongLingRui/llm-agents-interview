<img width="430" height="466" alt="image" src="https://github.com/user-attachments/assets/853fd99a-c0aa-46e1-bdb0-16db4aad0ca2" />



<img width="728" height="536" alt="image" src="https://github.com/user-attachments/assets/4c33ad50-b916-48f4-85b0-e6743eb43a73" />

<img width="375" height="198" alt="image" src="https://github.com/user-attachments/assets/639ec45d-772e-4430-9d22-35fec5204089" />


<img width="371" height="293" alt="image" src="https://github.com/user-attachments/assets/dad1bf2c-6961-43dd-8327-381a335f5d7d" />


<img width="686" height="645" alt="image" src="https://github.com/user-attachments/assets/1a2c4784-dc8f-4580-96ac-1fc51c71c776" />


<img width="674" height="767" alt="image" src="https://github.com/user-attachments/assets/495c9c22-12d6-4b2a-bbd2-cbf9f88e58c8" />


<img width="757" height="786" alt="image" src="https://github.com/user-attachments/assets/c9b5fbae-34e8-4ef3-9148-b08d32d7c542" />


### 内容提取

- **FLOPS 定义**：每秒浮点运算次数，是计算速度的衡量指标。
- **案例：GPT-4 训练算力需求**
  - 需 1330 亿千兆 FLOPS（即 \(1.33 \times 10^{26}\) FLOPS）。
  - 若使用 A100 的 FP16 Tensor Core（624 TFLOPS，即 \(6.24 \times 10^{14}\) 次/秒），且 OpenAI 用 25,000 台 A100，则训练时间为：
    \[
    \frac{1.33 \times 10^{26}}{6.24 \times 10^{14} \times 25,000} = 8525641 \text{秒} \approx 98.67 \text{天}
    \]

- **Table 2：H100 相对于 A100 的性能提升（单位：TFLOPS，\(10^{12}\) FLOPS）**

| 计算类型          | A100       | A100 Sparse | H100 SXM5  | H100 SXM5 Sparse | H100 SXM5 相对于 A100 的提升 |
|-------------------|------------|-------------|------------|------------------|------------------------------|
| FP8 Tensor Core   | NA         | NA          | 1978.9     | 3957.8           | 6.3x（vs A100 FP16 TC）|
| FP16              | 78         | NA          | 133.8      | NA               | 1.7x                         |
| FP16 Tensor Core  | 312        | 624         | 989.4      | 1978.9           | 3.2x                         |
| BF16 Tensor Core  | 312        | 624         | 989.4      | 1978.9           | 3.2x                         |
| FP32              | 19.5       | NA          | 66.9       | NA               | 3.4x                         |
| TF32 Tensor Core  | 156        | 312         | 494.7      | 989.4            | 3.2x                         |
| FP64              | 9.7        | NA          | 33.5       | NA               | 3.5x                         |
| FP64 Tensor Core  | 19.5       | NA          | 66.9       | NA               | 3.4x                         |
| INT8 Tensor Core  | 624 TOPS   | 1248 TOPS   | 1978.9     | 3957.8           | 3.2x                         |

*注：TOPS 为每秒万亿次整数运算，此处表格中 INT8 Tensor Core 列的单位为 TOPS，其余为 TFLOPS。*

