# MySkills

个人技能（skill）仓库，按主题分类存放。

## 目录结构

| 目录 | 用途 |
|------|------|
| [cesium/](cesium/) | CesiumJS 专属（地球渲染、3D Tiles、地形、材质） |
| [gis/](gis/) | GIS 通用（坐标系转换、SHP/MVT、地图服务、分析） |
| [3d/](3d/) | 其他 3D 开发（Three.js、Bevy、模型格式转换） |
| [rust/](rust/) | Rust 开发（引擎、工具、库） |
| [web/](web/) | Web 开发（前端、后端、API） |
| [devops/](devops/) | 运维部署（Docker、服务器、CI/CD、监控） |
| [ai/](ai/) | AI / LLM（提示词、Agent、模型调用） |
| [productivity/](productivity/) | 效率工具（文件管理、自动化、日常助手） |
| [templates/](templates/) | 技能模板（SKILL.md 骨架、示例） |

## Skill 命名规范

- 目录名使用 kebab-case（如 `cesium-tile-refresh`）
- 每个 skill 必须包含 `SKILL.md`（参考 [templates/](templates/)）
- 附带资源放 `references/`、`scripts/` 子目录

## 如何添加新 skill

1. 选择合适主题目录
2. 创建 `<skill-name>/SKILL.md`
3. 按模板填写描述、用法、示例
