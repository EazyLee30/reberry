# 博客布局与分享预览

三种语言页面共享 `site/style.css`，包含桌面侧栏、平板/手机顶部目录、流式字号、窄屏表格滚动、长命令换行及键盘焦点样式。

分享素材：

- `share-cover.svg`：由仓库原始键盘插画排版而成的封面源文件。
- `share-cover-v1.png`：1200 × 630 的分享封面，用于 Open Graph 与 X large-image card；不含设备照片或账户信息。
- `share-icon.png`：512 × 512 的莓红字标，用于方形缩略图线索。
- `favicon-32.png`、`apple-touch-icon.png`：浏览器与 Apple 分享/主屏幕图标线索。

所有语言都有独立标题、说明、canonical、og:url 和 og:locale，共用绝对 HTTPS 图片地址。图片使用版本化文件名，更新封面时应增加版本以减少旧图缓存。

## 验证边界

公开 HTTP 请求能证明页面及图片可抓取，不能证明每个平台一定展示同样的卡片。X 和微信最终效果仍需在对应客户端实际分享验证；本次不代用户发帖。

微信从 Safari 系统分享、微信内网页分享和复制链接等入口可能有不同的缩略图处理。favicon、Apple touch icon、image_src 与 Open Graph 是静态页面能提供的线索，不能保证 Safari 分享到朋友圈一定采用指定封面。精确控制微信内分享通常还涉及已认证公众号、域名配置和 JS-SDK 签名服务；本站本次未接入，不把静态元信息写成已完成微信 JS-SDK。

重新分享时先重新打开页面，必要时用新的查询参数触发新 URL 抓取。查询参数是客户端缓存排查办法，不保证清除平台服务器缓存。旧的已发布卡片不一定会更新。

## 发布源

`reberry` 仓库保留网站源文件 `site/`，实际 `https://eazylee.xyz/reberry/` 由 `EazyLee30/eazylee30.github.io` 的 `reberry/` 发布。修改后应同步该目录，并确认 GitHub Pages 工作流完成。不能只推送源仓库就宣称线上页面已更新。
