# Dive Into Deep Learning - Note

## 预备知识

### 数据操作

tensor：多维数组

tensor 的创建：

```py
import torch

# 创建 0...12 的 tensor
x = torch.arange(12)
> tensor([0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11])

# 获得 tensor 沿每个坐标轴的形状
x.shape
> torch.Size([12])

# 改变 tensor 形状
x.reshape(3, 4)
x
> tensor([[0,  1,  2,  3],
          [4,  5,  6,  7],
          [8,  9, 10, 11]])

# 用全 0、全 1 初始化矩阵
torch.zeros((2,3,4))
torch.zeros((2,3,4))
torch.randn(3,4)    # 每个元素从均值 0、标准差 1 的正态分布采样

# 指定初始值
torch.tensor([[2, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]])
```

tensor 的运算：

```py
# 按元素运算
torch.exp(x)

# 拼接 tensor，可以按不同维度
X = torch.arange(12, dtype=torch.float32).reshape((3,4))
Y = torch.tensor([[2.0, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]])
torch.cat((X, Y), dim=0), torch.cat((X, Y), dim=1)

# 生成二维 tensor，根据每个位置的比较为 true/false
x == y

# tensor 所有元素求和
x.sum()

# 广播机制：形状不同的 tensor 计算
a, b
> (tensor([[0],
           [1],
           [2]]),
   tensor([[0, 1]]))
a + b
> tensor([[0, 1],
          [1, 2],
          [2, 3]])
```

节省内存：

下面的命令可能会分配新的内存：`x = x + y`，为了避免这样，可以使用 `x += y` 或者用切片赋值

```py
Z = torch.zeros_like(Y)
print('id(Z):', id(Z))
Z[:] = X + Y
print('id(Z):', id(Z))
```

### 数据预处理

通常用 `pandas`

```py
# 拆分从 csv 读的数据，用平均值缺失值 NaN
inputs, outputs = data.iloc[:, 0:2], data.iloc[:, 2]
inputs = inputs.fillna(inputs.mean())
print(inputs)

# 将其中一列拆分为 xxx_<value> 和 xxx_nan
inputs = pd.get_dummies(inputs, dummy_na=True)
print(inputs)
>    NumRooms  Alley_Pave  Alley_nan
  0       3.0           1          0
  1       2.0           0          1
  2       4.0           0          1
  3       3.0           0          1

# 转换为tensor格式
x = torch.tensor(inputs.to_numpy(dtype=float))
y = torch.tensor(outputs.to_numpy(dtype=float))
X, y
```

### 线性代数
