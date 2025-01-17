+++
date = '2025-01-17T15:28:38+08:00'
draft = false
title = '如何finetune'
tags = ['Deep Learning']
summary = "最近为了加快模型训练，学习了如何finetune"
[cover]
    image = 'https://img-s-msn-com.akamaized.net/tenant/amp/entityid/BB1msMpz.img'
+++
# 什么是finetune
通义给出的概念：
> Fine-tuning（微调）是一种在机器学习和深度学习中常见的迁移学习技术。它指的是将一个已经在大型数据集上训练好> 的模型（通常称为预训练模型）应用于新的、但相关的问题或数据集上的过程。通过微调，可以利用预训练模型中已经学> 到的特征表示能力，并根据新任务的具体需求进行适当的调整。
>

个人觉得，只要是利用之前模型的权重，修改了网络结构或者训练数据等，为了加快训练，都可以叫做finetune。
# 如何finetune
既然要finetune，就要结合自己的问题具体去做。我的问题是，我有之前训练的网络权重，现在修改了网络结构，要利用之前的权重加速训练，那么就涉及两部分网络：
1. 之前的网络权重，称为```pretrained_params```
2. 新增的网络权重，称为```net_only_params```  
有两种思路，
1. 一种是冻结```pretrained_params```的权重，即通过设置```requires_grad=False```，只保证新增或修改的网络结构权重是可学习的，达到一定epoch时，将```pretrained_params```设置为```requires_grad=True```
2. 一种是将```pretrained_params```的学习率设置为很小，将```net_only_params```的学习率设置为较大，达到一定epoch时，设置为相等的
因为我的网络不支持设置```requires_grad=False```，所以只能采用第二种方法。
# 代码实现
```python
# 1. 增加参数，读取预训练权重
if args.init:  # load from previous checkpoint
        print(f"init {args.init} to device:{args.local_rank} ")
        init = torch.load(args.init, map_location=f"cuda:{args.local_rank}")
        init_state_dict = init["state_dict"]
        net_state_dict = net.state_dict()
        pretrained_dict = {k: v for k, v in init_state_dict.items() if k in net_state_dict and net_state_dict[k].size() == v.size()}
        # 更新模型的state_dict
        net_state_dict.update(pretrained_dict)
        net.load_state_dict(net_state_dict)
        # optimizer, init_lr_sch = configure_pretrained_optimizers(net, init, args)
        optimizer = configure_pretrained_optimizers(net, init, args)
        lr_scheduler = None
# 2. 设置优化器
def configure_pretrained_optimizers(net,init,args):
    init_params=[]
    net_only_params = []
    for name, param in net.named_parameters():
        if not name.endswith(".quantiles") and param.requires_grad:
            if name in init["state_dict"]:
                print(f"Pretrained Layer: {name}")
                init_params.append(param)
            else:
                print(f"New Layer: {name}")
                net_only_params.append(param)
            
    init_lr = args.learning_rate*0.0001
    net_lr = args.learning_rate 
    print(f"pretrained layer lr:{init_lr}, added layer lr:{net_lr}")
    param_groups = [
        {"params": init_params, "lr": init_lr},
        {"params": net_only_params, "lr": net_lr},
    ]
    optimizer = optim.Adam(param_groups)
    
    return optimizer #, get_init_sch(optimizer, args)
# 3. 到达一定epoch，设置两部分学习率相同，同时启动lr_scheduler
 if args.init:
    if epoch == args.equalize_epoch:
        for param_group in optimizer.param_groups:
            param_group['lr'] = args.learning_rate * 0.1
        lr_scheduler= optim.lr_scheduler.MultiStepLR(
            optimizer,
            milestones=milestones,
            gamma=0.5,
            last_epoch=-1
        )
```
