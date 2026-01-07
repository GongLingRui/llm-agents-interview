<img width="536" height="431" alt="image" src="https://github.com/user-attachments/assets/b47d0cc6-2cde-43e9-b488-a2348d6cbf0c" />


以下是提取的内容：

### 文字说明
Dice loss更关注预测结果与真实结果的重叠程度，而不是整体的分类准确度，可以很好的解决样本不平衡问题。


### Python代码
```python
import torch

def dice_coefficient(y_true, y_pred, smooth=1e-2):
    # Dice = (2*|X ∩ Y|)/ (|X| + |Y|)
    # ref: https://arxiv.org/pdf/1606.04797.pdf
    y_true = y_true.view(-1)
    y_pred = y_pred.view(-1)
    intersection = (y_true * y_pred).sum()
    return (2. * intersection + smooth) / (y_true.pow(2).sum() + y_pred.pow(2).sum() + smooth)

def dice_loss(y_true, y_pred):
    return 1 - dice_coefficient(y_true, y_pred)
```




