---
name: rms-gpu-query
description: Query and statistics for GPU servers in resource pools. Use this skill whenever the user needs to query GPU resources, count deliverable machines, generate resource reports, check X10/X40/X50/X60 GPU server inventory, or requests to see machine lists by rack nodes like rack.deliver/rack.stress/rack.uninstallos/rack.check. This skill provides both detailed server lists and statistical summaries for GPU infrastructure management.
---

# RMS GPU Query

Query and统计分析资源池内的 GPU 服务器信息。

## 依赖

依赖 `skills/rms-flow/` 包中的 ServerQuery 模块来执行 RMS 查询 API 调用。

## 核心查询能力

### 支持的 GPU 型号

- X10*8
- X40*8
- X50*8
- X60*8

### 支持的节点

- rack.deliver - 交付节点
- rack.stress - 压测节点
- rack.uninstallos - 卸载节点
- rack.check - 检查节点
- rack.flow.repair - 维修节点（通常与其他节点叠加）

### 支持的查询操作符

- `eq` - 等于
- `in` - 在数组中
- `leftlike` - 左模糊匹配（支持 * 通配符）

## 查询步骤

### 1. 初始化查询客户端

```python
import sys
import os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), 'skills/rms-flow'))
from query import ServerQuery

sq = ServerQuery()
```

### 2. 构建查询条件

查询使用 `_query()` 方法，传入条件列表。每个条件包含：
- `key`: 字段名
- `operate`: 操作符（eq/in/leftlike）
- `value`: 值

**常用查询字段：**
- `sn` - 服务器 SN
- `hostname` - 主机名
- `idcName` - 机房名称
- `type` - 类型（physical 为物理机）
- `gpuModel` - GPU 型号
- `suitName` - 套餐名称
- `nodes` - 节点列表

**基础查询示例：**

```python
result = sq._query(
    condition=[
        {'key': 'nodes', 'operate': 'in', 'value': ['rack.deliver', 'rack.stress']},
        {'key': 'type', 'operate': 'eq', 'value': 'physical'},
        {'key': 'gpuModel', 'operate': 'in', 'value': ['X10*8', 'X40*8', 'X50*8', 'X60*8']}
    ],
    fields='assetPtrId,sn,hostname,gpuModel,suitName,idcName,nodes',
    page=1,
    page_size=1000
)
```

### 3. 客户端过滤（如果需要）

某些过滤条件需要客户端处理，因为 API 可能不支持：
- 排除特定机房（如 usewr01）
- 过滤特定节点组合（如同时包含 rack.flow.repair）

```python
# 剔除 usewr01 机房
servers = [s for s in result['data'] if s.get('idcName') != 'usewr01']

# 只保留单一节点（不含 rack.flow.repair）
single_node_servers = [
    s for s in servers
    if 'rack.deliver' in s.get('nodes', [])
    and 'rack.flow.repair' not in s.get('nodes', [])
]
```

## 统计分析

### 单一节点统计

只统计仅包含指定节点的服务器。

**统计要点：**
- 遍历服务器列表
- 检查 `nodes` 字段
- 确保不包含 `rack.flow.repair`
- 按 GPU 型号分组统计

### 叠加节点统计

统计同时包含目标节点和 `rack.flow.repair` 的服务器。

**统计要点：**
- 检查服务器是否包含 rack.flow.repair
- 检查是否同时包含指定节点
- 按节点和 GPU 型号交叉统计

### 分组统计

使用 `defaultdict` 进行分组：

```python
from collections import defaultdict

# {节点: {gpu_model: count}}
stats = defaultdict(lambda: defaultdict(int))

for server in servers:
    node_list = server.get('nodes', [])
    gpu_model = server.get('gpuModel')

    for node in target_nodes:
        if node in node_list:
            stats[node][gpu_model] += 1
```

## 输出格式

### 详细服务器列表

表格格式，包含以下字段：
- SN
- GPU 型号
- 套餐名称
- 机房
- 节点

### 汇总表格

交叉统计表，按节点和 GPU 型号：
- 行：节点（ rack.deliver、rack.stress 等）
- 列：GPU 型号（X10*8、X40*8、X50*8、X60*8）
- 单元格：机器数量

### SN 列表

纯文本格式，每行一个 SN，方便复制使用。

## 输出示例

### 基础查询输出示例

```
资源池内 X10-X60 GPU 服务器统计（已剔除 usewr01 机房）

总计: 9 台

【X50*8】
  小计: 6 台
  套餐分布:
    GE151: 6 台

【X60*8】
  小计: 2 台
  套餐分布:
    GE131: 2 台
```

### 节点汇总表示例

```
| 节点     | X10*8 | X40*8 | X50*8 | X60*8 | 总计 |
|----------|-------|-------|-------|-------|------|
| rack.deliver | -    | -     | 6     | 2     | 8    |
| rack.stress  | 1    | -     | -     | 1     | 2    |
| 列总计       | 1    | 0     | 6     | 3     | 10   |
```

## 常见使用场景

1. **查询可交付的 GPU 机器**：查询 rack.deliver 节点下的特定 GPU 型号
2. **统计资源池机器分布**：按节点和 GPU 型号统计所有机器
3. **生成资源报告**：生成详细列表和汇总表格
4. **获取 SN 列表**：提供可复制的 SN 列表供后续使用
5. **筛选特定机房**：排除或包含特定机房
6. **分析维修状态**：统计叠加 repair 节点的机器

## 注意事项

- API 的 `neq` 操作符可能不支持，建议在客户端过滤
- 查询结果需要检查 `success` 字段
- 每页最大记录数取决于 API 配置，通常为 1000
- 服务器操作是高危操作，统计查询仅供参考
