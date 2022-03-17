# paddle.nn.Softmax2D 设计文档

|API名称 | paddle.nn.Softmax2D | 
|---|---|
|提交作者<input type="checkbox" class="rowselector hidden"> | xiaoyangyang2 | 
|提交时间<input type="checkbox" class="rowselector hidden"> | 2022-03-17 | 
|版本号 | V1.0 | 
|依赖飞桨版本<input type="checkbox" class="rowselector hidden"> | v2.2.0 | 
|文件名 | 20220317_api_design_for_Softmax2D.md<br> | 

# 一、概述

## 1、相关背景
为了提升飞桨API丰富度，Paddle需要扩充API`paddle.nn.Softmax2D`。
## 2、功能目标
增加API`paddle.nn.Softmax2D`，实现针对 3D 或者 4D 的 tensor 在空间维度计算softmax，从而输出 tensor 在每个空间维度（channels, hj, wj）的 tensor 求和为1。

## 3、意义
飞桨支持计算Softmax2D

# 二、飞桨现状
目前paddle缺少相关功能实现。

API方面，已有类似功能的API，paddle.nn.Softmax 可以通过指定 axis 来完成对空间维度的 softmax 计算，但是不如直接调用 paddle.nn.Softmax2D 优雅。


# 三、业内方案调研
## Pytorch
Pytorch中有API`torch.nn.Softmax2d(input: Tensor) -> Tensor`.

在pytorch中，介绍为：

> Applies SoftMax over features to each spatial location.
> 
> When given an image of `Channels x Height x Width`, it will apply `Softmax` to each location :math:`(Channels, h_i, w_j)`.


### 实现方法
在实现方法上, Pytorch是通过指定 torch.nn.functional.softmax() 操作子的 dim=-3 来实现对空间维度的softmax计算，[代码位置](https://pytorch.org/docs/stable/_modules/torch/nn/modules/activation.html#Softmax2d)。



# 四、对比分析
- 使用场景与功能：在输入 tensor 的维度支持上，Pytorch只支持 (N, C, H, W)` or `(C, H, W)，如果输入是 (N, H, W, C)` or `(H, W, C) 则不能支持。


# 五、方案设计
## 命名与参数设计
API设计为`paddle.nn.Softmax2D(x, channel_last=False, name=None)`

- 命名与参数顺序为：形参名`input`->`x`, `channel_last=False`, 与paddle其他API保持一致性，不影响实际功能使用。
- 参数类型中，`channel_last`是`bool`类型，以同时支持(N, C, H, W)和(N, H, W, C)两种不同输入的场景。


## 底层OP设计
使用已有API实现，不再单独设计OP。

## API实现方案
主要按下列步骤进行实现，实现位置为 Paddle repo 的`python/paddle/nn/layer/activation.py`文件，与`softmax`等方法放在一起：
1. 首先判断输入 tensor 的维度，只能为3D 或 4D；
2. 初始化私有变量
3. 若 `channel_last` 为 True，则输入 tensor 的空间维度在最后面，需要更新_axis；
4. 使用`paddle.nn.functional.softmax()` 对指定的`axis`进行softmax计算；
5. 返回与输入相同形状的tensor。


 
# 六、测试和验收的考量
测试考虑的case如下：

- 和Pytorch结果的数值的一致性、真实输出与逻辑输出是否一致，即功能是否正常；
- 输入含`NaN`结果的正确性；
- 错误检查：输入tensor维度不是3D 或 4D时能正确抛出错误；

# 七、可行性分析及规划排期

方案主要依赖现有paddle api组合而成，工期上可以满足在当前版本周期内开发完成。

# 八、影响面
为独立新增API，对其他模块没有影响



# 名词解释
无
# 附件及参考资料
无
