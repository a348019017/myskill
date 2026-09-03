# MySkills

个人技能（skill）仓库，按功能领域分类存放。

## 目录结构

| 目录 | 用途 |
|------|------|
| [3d-gis/](3d-gis/) | 3D 开发 / GIS 相关（Cesium、Three.js、地图、模型处理） |
| [coding/](coding/) | 编程开发类（语言、框架、代码生成、调试） |
| [devops/](devops/) | 运维部署类（Docker、服务器、CI/CD、监控） |
| [ai-ml/](ai-ml/) | AI / 机器学习相关（LLM、提示词、模型工具） |
| [productivity/](productivity/) | 效率工具类（文件管理、自动化、日常助手） |
| [templates/](templates/) | 技能模板（SKILL.md 骨架、示例技能） |

## Skill 命名规范

- 目录名使用 kebab-case（如 `cesium-tile-refresh`）
- 每个 skill 必须包含 `SKILL.md`（参考 [templates/](templates/)）
- 附带资源放 `references/`、`scripts/` 子目录

## 如何添加新 skill

1. 选择合适分类目录
2. 创建 `<skill-name>/SKILL.md`
3. 按模板填写描述、用法、示例
