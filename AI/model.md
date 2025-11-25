<img width="338" height="463" alt="image" src="https://github.com/user-attachments/assets/6bb26c7f-fc72-40fa-bcad-582d329ad945" />


```
import torch
import torch.nn as nn
from torch.nn import functional as F
import math
# Hyperparameters
d_model = 512
context_length = 16
num_heads = 8
head_size = d_model // num_heads
device = 'cuda' if torch.cuda.is_available() else 'cpu'
dropout = 0.1
max_token_value = 1000256
num_blocks = 12
class FeedForwardNetwork(nn.Module):
    def _init_(self):
        super().__init__()
        self.ffn = nn.Sequential(
            nn.Linear(d_model,4*d_model),
            nn.ReLU(),
            nn.Linear(4*d_model,d_model),
            nn.Dropout(dropout)
        )
    def forward(self,x):
        return self.ffn(x)

class Attention(nn.Module):
    def __init__(self):
        super().__init__()
        self.Wq = nn.Linear(d_model,head_size)
        self.Wk = nn.Linear(d_model,head_size)
        self.Wv = nn.Linear(d_model,head_size)
        self.register_buffer('mask',torch.tril(torch.ones(context_length,context_length)))
        self.dropout = nn.Dropout(dropout)
    def forward(self,x):
        B,T,D = x.shape
        q = self.Wq(x)#q:[batch_size,context_length,head_size]
        k = self.Wk(x)
        v = self.Wv(x)
        output = (q @ k.transpose(-2,-1))/math.sqrt(head_size)
        output.masked_fill(self.mask[:T,:T]==0,float('-inf'))
        output = F.softmax(output)
        output = self.Dropout(output)#optianla
        output = output @ v

        return output

class MultiHeadAttention(nn.Module):
    def __init__(self):
        super().__init__()
        self.heads = self.ModuleList([Attention() for _ in range(num_heads)])
        self.proj = nn.Linear(d_model,d_model)
        self.Wo = nn.Linear(d_model,d_model)
        self.dropout = nn.Dropout(dropout)
        
    def forward(self,x):
        output = torch.cat([head(x) for head in self.heads],dim=-1)
        output = self.proj(output)
        output = self.Wo(output)
        output = self.dropout(output)
        return output


class TransformerBlock(nn.Module):
    def __init__(self):
        super().__init__()
        self.ln1 = nn.LayerNorm(d_model)
        self.ln2 = nn.LayerNorm(d_model)
        self.mha = MultiHeadAttention()
        self.ffn = FeedForwardNetwork()
    def forward(self,x):
        x = x +self.mha(self.ln1(x))
        x = x+ self.ffn(self.ln2(x))
        return x
class Model(nn.Module):
    def __init__(self,max_token_value = 10082):
        super().__init__()
        self.vocab_linear = nn.Linear(max_token_value,d_model)
        self.te_lookup_table = nn.Embedding(max_token_value,d_model)
        # token embedings
        self.transformers_block = nn.Sequential(
            [TransformerBlock() for _ in range(num_blocks)] +nn.Layernorm(d_model)
        )
    def forward(self,x_batch,y_batch=None):
        B,T,D = x_batch.shape
        pre_lookup_table = torch.zeros(context_length,d_model,device=device)
        position = torch.arange(0,context_length,dtype =torch.float).unsqueeze(1)
        torch.arrange(0,d_model,2).float().d_model
        div_term = torch.exp(-math.log(10000.0)*torch.arange(0,d_model,2).float()/d_model * position)
        pre_lookup_table[:,0::2] = torch.sin(position*div_term)
        pre_lookup_table[:,1::2] = torch.cos(position*div_term)
        output   = self.te_lookup_table(x_batch)+pre_lookup_table
        output = self.transformers_block(output)
        output = self.vocab_linear(output)
        logits = self.vocab_linear(output)
        if y_batch  is not None:
            B,T,D= logits.shape
            logits_reshaped = torch.view(B*T,D)
            y_batch_reshaped = torch.view(B*T)
            loss = F.cross_entropy(input = logits_reshaped,target = y_batch_reshaped)

        else:
            loss = None
            return logits,loss
def generate(self,x_batch,max_new_tokens = 100,temperatire = 1.0):
    pass
```

