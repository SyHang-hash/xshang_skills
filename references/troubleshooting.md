# 问题排查与踩坑手册

从 194 次真实对话中提炼的高频问题和解法。

## 排查原则

1. **先看报错，再看代码** — 不要看到报错就猜，读完整错误信息
2. **改一处测一处** — 不要一口气改 5 个文件然后看效果
3. **同一个 bug 修 3 轮还没好** — 停下来换方案，不要在破方案上打补丁
4. **降级优先** — 某个依赖持续出问题，先降版本，锁定稳定版
5. **后端数据先 console.log** — 前端显示异常 80% 是数据结构不匹配

## 高频错误速查表

### 模块导出相关

| 错误 | 根因 | 解法 |
|------|------|------|
| `does not provide an export named 'xxx'` | barrel export（api/index.js）没更新 | 检查 `api/index.js` 的 re-export 是否包含新函数 |
| `"init" is not exported from 'echarts'` | 从 echarts 直接导入而非按需导入 | 用 `@/utils/echarts` 导入，不要从 `echarts` 包直接导 |

### HTTPS / 混合内容

| 错误 | 根因 | 解法 |
|------|------|------|
| `Mixed Content: loaded over HTTPS but requested HTTP` | 前端 HTTPS，后端 API 是 HTTP | Nginx 反向代理统一走 HTTPS，或后端也上 SSL |

### Mermaid 渲染（高频重灾区）

| 错误 | 根因 | 解法 |
|------|------|------|
| `Parsing failed: unexpected character` | 中文字符未用引号包裹 | 节点 ID 用英文，文本用引号：`A["数据结构"]` |
| `Expecting token of type 'ID' but found...` | 用了 beta 图表类型或语法不兼容 | 换稳定类型（flowchart, sequence），避免 radar-beta, xychart-beta |
| 渲染空白 | Mermaid 初始化时机不对 | 确保 DOM 挂载后再初始化，用 nextTick |
| 中文标签持续报错 | Mermaid 对中文兼容性差 | 如果修 2 轮没好，果断切换到 ECharts 图表 |

**Mermaid 使用黄金规则**：
- 节点 ID 必须英文，中文放引号里
- 优先用 flowchart、sequence、gantt，不用 beta 功能
- 锁定一个稳定版本不要随便升级
- 修 2 轮没好就换 ECharts

### PDF 导出

| 错误 | 根因 | 解法 |
|------|------|------|
| `Unable to find element in cloned iframe` | html2canvas 克隆 DOM 时找不到元素 | 确保 DOM 完全渲染：等 nextTick + 适当延迟 |
| 导出白屏 | 动态内容（图表/Mermaid SVG）未渲染完 | 等所有异步渲染完成后再触发导出 |
| 样式丢失 | 跨域字体/图片 | html2canvas 配置 `useCORS: true` |

**PDF 导出原则**：不要用浏览器打印接口（window.print），用 html2canvas 截图。

### 构建相关

| 错误 | 根因 | 解法 |
|------|------|------|
| `Top-level await is not available` | 构建目标太低 | vite config 加 `target: 'esnext'` |
| `Failed to parse source for import analysis` | 依赖包语法不兼容 | 检查依赖版本，降级或配置 optimizeDeps |

### Nginx / 部署

| 错误 | 根因 | 解法 |
|------|------|------|
| `open() sites-enabled/xxx failed` | Nginx 配置文件路径错误 | 检查 `nginx.conf` 的 `include` 路径，确认文件存在 |
| SPA 刷新 404 | 没配 `try_files` | 加 `try_files $uri $uri/ /index.html` |
| SSL 证书安装失败 | Certbot 找不到 Nginx 配置 | 确保 server_name 匹配域名 |

### Electron 打包

| 错误 | 根因 | 解法 |
|------|------|------|
| `app.asar being used by another process` | 上次打包的进程还在跑 | 关闭所有 Electron 进程后重试 |
| 便携版打开白屏 | base 路径不对 | vite config 加 `base: './'` |

### 数据显示

| 错误 | 根因 | 解法 |
|------|------|------|
| 新用户登录有残余数据 | 没做空数据判断 | 先查用户是否有记录，无记录不渲染图表 |
| 数据格式与预期不符 | 后端返回结构变了 | console.log 看实际结构，对照接口文档 |
| 图表数据不显示 | ECharts setOption 时数据还没加载完 | 数据 ready 后再 initChart，用 nextTick |

## "修了多轮还在错"的处理流程

```
1. 停下来，不要继续在原方案上打补丁
2. 回到最简方案：先保证基础功能能跑
3. 如果某个依赖持续出问题 → 降级版本或换方案
4. 告诉用户："这个方案不稳定，建议换成 XXX"
5. 用新方案重新开始，不要在旧代码上缝补
```
