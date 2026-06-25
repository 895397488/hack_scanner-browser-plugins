# hack_scanner-browser-plugins
溯源仓库hack_scanner的浏览器扩展插件版
# Hack Scanner — Browser Extension

> 将 Python hack 扫描器的核心能力迁移到浏览器，实现实时 + 手动的 Web 安全检测。

## 安装
（这里以Microsoft edge作为演示）
1. 打开 Edge 浏览器，访问 `edge://extensions/`
2. 开启左下角 **"开发者模式"**
3. 点击 **"加载已解压的扩展程序"**
4. 选择本目录 (`hack-extension/`)

## 使用方式

- **实时检测**：启用"自动模式"后，每次访问网站时自动执行安全扫描
- **手动扫描**：切换到"手动模式"，点击工具栏中的扩展图标 → 点击 "扫描" 按钮
- **导出报告**：点击 "导出 JSON" 保存扫描结果

## 功能清单

| 检测类别 | 说明 |
|---------|------|
| HTTP 安全头 | X-Frame-Options / CSP / HSTS / X-Content-Type-Options 等 9 项检查 |
| SSL/TLS | HTTPS vs HTTP 协议检测 |
| XSS | DOM 回显检测 + URL 参数反射型 XSS + 存储型 XSS |
| SQLi | URL 参数错误基注入测试 |
| CORS | Wildcard / Origin 反射 / Credentials 配置检测 |
| SSRF | 内网 IP 重定向检测 (URL/redirect/target/path/src params) |
| HTTP 方法 | TRACE / PUT / DELETE 支持检测 |
| 技术栈指纹 | React/Vue/Angular/WordPress/Cloudflare/Nginx/Apache 等 50+ 技术识别 |

## Manifest V3 权限说明

| Permission | 用途 |
|-----------|------|
| `activeTab` | 读取当前标签页 URL |
| `storage` | 保存自动/手动模式设置 |
| `webNavigation` | 监听页面导航事件触发自动扫描 |
| `declarativeContent` | 扩展图标显示控制 |
| `scripting` | Content Script 注入（已声明匹配所有 URL） |
| `<all_urls>` | 对所有网站执行安全检测 |

## 架构说明

```
hack-extension/
├── manifest.json              # Manifest V3 配置
├── background.js              # Service Worker — 后台扫描引擎
├── content.js                 # Content Script — 页面 DOM 分析
├── popup/
│   ├── index.html             # Popup UI (暗色黑客风)
│   ├── popup.css              # Dark theme styles
│   └── popup.js               # Popup logic + scan orchestration
├── lib/
│   ├── headers.js             # HTTP 安全头检测模块
│   ├── xss_detector.js        # XSS Payload 生成与测试
│   ├── sqli_detector.js       # SQLi 注入测试
│   ├── fingerprint.js         # Wappalyzer 兼容技术栈指纹库
│   ├── cors_detector.js       # CORS 配置检测 + HTTP 方法安全
│   ├── ssrf_detector.js       # SSRF Payload 发送
│   └── report.js              # JSON/HTML 报告导出
└── icons/
    ├── icon16.png
    ├── icon48.png
    └── icon128.png
```

## 注意事项

- 本工具仅限**授权安全评估**使用
- 浏览器扩展的扫描能力受限于 CSP 和同源策略，无法替代原生工具（如 hack scanner）的全部功能
- 文件扫描、目录爆破等需要文件系统访问的能力无法在扩展中实现
- SSRF/SQLi/XSS 检测为被动 + 启发式分析，误报率较高，需人工验证

## 从 Python hack 迁移的映射关系

| hack (Python) | Extension (JS) |
|--------------|----------------|
| `url_scanner.py` HTTP头检测 | `lib/headers.js` analyzeHeaders() |
| `url_scanner.py` XSS检测 | `lib/xss_detector.js` detectReflectedXSS() |
| `url_scanner.py` SQLi检测 | `lib/sqli_detector.js` detectSQLi() |
| `shannon_context.py` 技术栈指纹 | `lib/fingerprint.js` getFullFingerprint() |
| `url_scanner.py` CORS检查 | `lib/cors_detector.js` detectCORSMisconfig() |
| `url_scanner.py` SSRF检测 | `lib/ssrf_detector.js` detectSSRF() |
| `hack_scanner.py` 报告生成 | `lib/report.js` ScanReport class |
