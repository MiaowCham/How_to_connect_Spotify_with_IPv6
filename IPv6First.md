# **使 Windows 全局优先使用 IPv6 连接**
> [!note]
> 本文档由 DeepSeek 总结并生成

要让 Windows **所有网络连接（包括 `ping`、浏览器、游戏等）优先使用 IPv6**，请按以下步骤操作：

---

## **📌 核心步骤（推荐）**
### **1. 修改 IPv6 前缀策略（强制优先级）**
**以管理员身份运行 CMD/PowerShell**，执行：
```cmd
netsh interface ipv6 set prefixpolicy ::/0 100 0 persistent
netsh interface ipv6 set prefixpolicy ::ffff:0:0/96 40 1 persistent
netsh interface ipv6 set global randomizeidentifiers=disabled
netsh interface ipv6 set global randomizeidentifiers=enabled
```
- **`::/0`**：代表所有 IPv6 地址，优先级设为 **100**（最高）
- **`::ffff:0:0/96`**：代表 IPv4 映射地址，优先级设为 **40**（低于 IPv6）
- 第三、四行通过**重新加载 IPv6 配置**，使策略立即生效。

### **2. 验证配置**
```cmd
netsh interface ipv6 show prefixpolicies
```
- 检查输出，确保 `::/0` 的 **Precedence** 为 **50**，且排在 `::ffff:0:0/96` 之前。

### **3. 验证连接**
可以在 `终端App` 中依次运行以下命令检测连接情况：
1. 打开 **命令提示符**（`Win + R` → `cmd` → 回车）
2. 运行：
    ```cmd
    ping api.spotify.com
    ping accounts.spotify.com
    ```
    如果无需 `-6` 参数即可 ping 通 IPv6，则证明已经完成设置

---

## **🔍 额外优化（可选）**

### **4. 检查 DNS 解析顺序**
```cmd
nslookup spotify.com
```
- 如果 IPv6（AAAA 记录）排在 IPv4（A 记录）之前，说明 DNS 策略已生效。

### **5. 测试 `ping` 和 `curl`**
```cmd
ping api.spotify.com          # 应返回 IPv6 地址
curl -v https://api.spotify.com  # 应使用 IPv6 连接
```

---

## **❓ 常见问题**
### **Q1: 为什么 `ping` 仍然走 IPv4？**
- 可能原因：
  - 目标网站没有 IPv6（AAAA 记录）。
  - 本地防火墙/ISP 拦截了 IPv6 ICMP 请求。
  - DNS 缓存未更新（尝试 `ipconfig /flushdns`）。

### **Q2: 如何恢复默认设置？**
```cmd
netsh interface ipv6 reset
```
- 重启计算机后，所有设置会恢复默认。
