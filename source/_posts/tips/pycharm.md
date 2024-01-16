---
title: pycharm小技巧
categories:
  - tips
tags:
  - pycharm
mathjax: true
---
<meta name="referrer" content="no-referrer"/>

<!--more-->

## 开启show comand queue(>>>)

![image-20231023163651677](https://gitee.com/hollis7/pictures/raw/master/2023/10/23/43666_image-20231023163651677.png)

## 查看模型结构

法一：

~~~python
print(model)
~~~

```
      (11): RobertaLayer(
        (attention): RobertaAttention(
          (self): RobertaSelfAttention(
            (query): Linear(in_features=768, out_features=768, bias=True)
            (key): Linear(in_features=768, out_features=768, bias=True)
            (value): Linear(in_features=768, out_features=768, bias=True)
            (dropout): Dropout(p=0.1, inplace=False)
          )
          (output): RobertaSelfOutput(
            (dense): Linear(in_features=768, out_features=768, bias=True)
            (LayerNorm): LayerNorm((768,), eps=1e-05, elementwise_affine=True)
            (dropout): Dropout(p=0.1, inplace=False)
          )
        )
        (intermediate): RobertaIntermediate(
          (dense): Linear(in_features=768, out_features=3072, bias=True)
          (intermediate_act_fn): GELUActivation()
        )
        (output): RobertaOutput(
          (dense): Linear(in_features=3072, out_features=768, bias=True)
          (LayerNorm): LayerNorm((768,), eps=1e-05, elementwise_affine=True)
          (dropout): Dropout(p=0.1, inplace=False)
        )
      )
    )
  )
  (pooler): RobertaPooler(
    (dense): Linear(in_features=768, out_features=768, bias=True)
    (activation): Tanh()
  )
)
```

法二：

~~~python
for name, parameters in model.named_parameters():
    print(name, ':', parameters.size())
~~~

```
encoder.layer.11.attention.self.query.bias : torch.Size([768])
encoder.layer.11.attention.self.key.weight : torch.Size([768, 768])
encoder.layer.11.attention.self.key.bias : torch.Size([768])
encoder.layer.11.attention.self.value.weight : torch.Size([768, 768])
encoder.layer.11.attention.self.value.bias : torch.Size([768])
encoder.layer.11.attention.output.dense.weight : torch.Size([768, 768])
encoder.layer.11.attention.output.dense.bias : torch.Size([768])
encoder.layer.11.attention.output.LayerNorm.weight : torch.Size([768])
encoder.layer.11.attention.output.LayerNorm.bias : torch.Size([768])
encoder.layer.11.intermediate.dense.weight : torch.Size([3072, 768])
encoder.layer.11.intermediate.dense.bias : torch.Size([3072])
encoder.layer.11.output.dense.weight : torch.Size([768, 3072])
encoder.layer.11.output.dense.bias : torch.Size([768])
encoder.layer.11.output.LayerNorm.weight : torch.Size([768])
encoder.layer.11.output.LayerNorm.bias : torch.Size([768])
pooler.dense.weight : torch.Size([768, 768])
pooler.dense.bias : torch.Size([768])
```

## 关闭占用端口的程序

查看占用端口的程序

~~~bash
netstat -ano | findstr 8080
~~~

使用命令关闭

~~~bash
taskkill -PID 进程号 -F
~~~

