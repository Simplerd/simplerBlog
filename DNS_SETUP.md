# GitHub Pages 自定义域名 DNS 配置指南

## 问题说明
GitHub Pages 提示 DNS 解析失败，域名 `simpler.org.cn` 没有正确解析到 GitHub Pages 服务器。

## 解决方案

### 方法一：使用 CNAME 记录（推荐）

在你的域名 DNS 提供商（如阿里云、腾讯云、Cloudflare 等）添加以下 DNS 记录：

**记录类型：** CNAME  
**主机记录：** @（或留空，表示根域名）  
**记录值：** `simplerd.github.io`  
**TTL：** 600（或默认值）

**注意：** 如果还需要支持 `www.simpler.org.cn`，需要额外添加：
- **记录类型：** CNAME
- **主机记录：** www
- **记录值：** `simplerd.github.io`
- **TTL：** 600

### 方法二：使用 A 记录

如果使用 A 记录，需要添加以下 4 条记录（GitHub Pages 要求）：

**记录类型：** A  
**主机记录：** @（或留空）  
**记录值：** 
- `185.199.108.153`
- `185.199.109.153`
- `185.199.110.153`
- `185.199.111.153`

需要分别添加这 4 条 A 记录。

### GitHub 仓库设置

1. 进入你的 GitHub 仓库：`https://github.com/Simplerd/simplerBlog`
2. 点击 **Settings** → **Pages**
3. 在 **Custom domain** 部分输入：`simpler.org.cn`
4. 勾选 **Enforce HTTPS**（可选，但推荐）

### 验证步骤

1. **检查 DNS 解析：**
   ```bash
   # 在终端运行以下命令检查 DNS 解析
   dig simpler.org.cn
   # 或
   nslookup simpler.org.cn
   ```

2. **等待 DNS 传播：**
   - DNS 更改通常需要几分钟到几小时才能生效
   - 可以使用在线工具检查：https://www.whatsmydns.net/

3. **检查 GitHub Pages 状态：**
   - 回到 GitHub 仓库的 Pages 设置页面
   - 等待绿色勾号出现，表示 DNS 配置正确

### 常见问题

**Q: 为什么还是显示 DNS 解析失败？**  
A: 
- 确认 DNS 记录已正确添加
- 等待 DNS 传播（最长可能需要 48 小时）
- 清除浏览器缓存或使用无痕模式访问
- 检查域名是否已过期

**Q: 支持 www 子域名吗？**  
A: 可以，需要额外添加 `www` 的 CNAME 记录指向 `simplerd.github.io`，并在 GitHub Pages 设置中添加 `www.simpler.org.cn` 作为备用域名。

**Q: 如何强制 HTTPS？**  
A: 在 GitHub Pages 设置中勾选 **Enforce HTTPS**，但需要先确保 DNS 配置正确且 GitHub 已检测到域名。

### 当前项目配置

- ✅ CNAME 文件已配置：`simpler.org.cn`
- ✅ Hexo 配置 URL：`https://www.simpler.org.cn`
- ⚠️ 需要配置 DNS 记录

### 下一步

1. 登录你的域名 DNS 提供商管理后台
2. 添加上述 DNS 记录
3. 在 GitHub 仓库设置中配置自定义域名
4. 等待 DNS 传播并验证

