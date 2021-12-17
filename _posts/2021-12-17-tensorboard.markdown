---
layout: post
title:  "tensorboard"
date:   2021-12-17 20:35:35 +0800
categories: command
---

TensorBoard setup
```
from torch.utils.tensorboard import SummaryWriter

# default `log_dir` is "runs" - we'll be more specific here
writer = SummaryWriter('runs/fashion_mnist_experiment_1')
```

Tracking model training with TensorBoard
```
writer.add_scalar('training loss', running_loss / 1000, epoch * len(trainloader) + i)
```

add hi
```
# histogram
for name, params in model.named_parameters():
    if params.requires_grad:
        writer.add_histogram(name, params, train_iteration)

```

run tensorboard
```
tensorboard --logdir=runs
```