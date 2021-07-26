# Studio版本更新说明
## v2.2.1（2021.05.06）

- 修复
  - 修复浏览器兼容问题
  - 修复 big int 类型数据渲染问题
  - 修复导入数据无法获取结果的问题

## v2.2.0 (2021.04.23)

- 功能增强:
  - 图探索
    - UI 改版
    - 新增操作面板，右键菜单，支持缩放、拓展、修改节点颜色等操作
    - 支持部分操作快捷键操作
    - 支持快速查找画板中相关节点
    - 拓展功能增强，支持多步查询
    - 新增图算法查询功能

- 修复: 
  - 控制台
    - 修复 MATCH 返回结构无法导向图探索的问题

## v2.1.9-beta (2021.03.24)

- 修复:
  - 导入
    - 支持导入 int 类型 vid 的数据

## v2.1.8-beta（2021.03.01）

- 功能增强:
  - 控制台
    - 历史语句支持清空
  - 图探索
    - 优化大量数据显示
    - 操作面板增加按钮支持缩放，拖拽功能

## v2.1.7-beta（2021.03.01）

- 功能增强:
  - 控制台
    - EXPLAIN dot 参数结果支持可视化

## v2.1.6-beta（2021.02.22）
- 修复:
  - 修复数字超过精度时显示不准确的问题

- 功能增强:
  - 图探索
    - 支持查询 int 类型的 vid 节点

## v2.1.5-beta（2021.01.25）

- 修复:
  - 图探索
    - 修复返回数据结构变化引起的渲染问题

## v2.1.4-beta（2021.01.19）

- 功能增强:
  - Schema
    - 支持 space/tag/edge/index 关于 fixed string 类型配置
  - 图探索
    - 选中节点后节点信息固定页面显示
    - 显示节点/边的配置持久化保存


## v2.1.3-beta（2021.01.12）

- 修复:
  - 图探索
    - 修复回退步骤时节点渲染问题
    - 拓展时添加默认最大限制数量

## v2.1.2-beta（2020.12.29）

- 功能增强:
  - 图探索
    - 支持查询结果导出图片
    - 支持双击节点快速拓展

## v2.1.1-beta（2020.12.23）

- 修复:
  - 导入
    - 修复 CSV 解析预览不正确的问题
    - 修复上传大容量文件被删减的问题

## v2.1.0-beta（2020.12.22）

- 功能增强:
  - 控制台
    - 支持 GET SUBGRAPH 导入子图
  - 导入
    - 支持 CSV 导入数据

- 修复
  - 控制台
    - 修复 int / double 列数据排序问题
  - 图探索
    - 修正 Vertex 属性显示问题

## v2.0.0-alpha（2020.11.24）

- 功能增强:
  - 适配 Nebula Graph v2.0.0-alpha