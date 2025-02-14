# EdgeLogGuard 用户手册 V1.0

---

## **目录**
1. [快速入门](#快速入门)
2. [系统安装](#系统安装)
3. [功能使用指南](#功能使用指南)
4. [安全配置](#安全配置)
5. [故障排查](#故障排查)
6. [技术支持](#技术支持)

---

## **1. 快速入门**
### 1.1 三分钟启动方案
**适用场景**：个人开发者首次部署
```bash
# 一键部署（支持Linux环境）
curl -sSL https://edgelogguard.com/install.sh | bash
```
**操作流程**：
1. 浏览器访问 `http://<服务器IP>:8080`，完成管理员账户注册
2. 在「日志源」页面添加首个日志文件路径（如 `/var/log/nginx/access.log`）
3. 前往「安全规则」启用默认攻击检测模板

---

## **2. 系统安装**
### 2.1 兼容性要求
| 环境         | 最低配置                           | 推荐配置                           |
|--------------|-----------------------------------|------------------------------------|
| **服务端**   | 2核 CPU / 4GB 内存 / 50GB 存储    | 4核 CPU / 16GB 内存 / 200GB SSD    |
| **边缘节点** | 1核 CPU / 1GB 内存 / 10Mbps 带宽  | 2核 CPU / 2GB 内存 / 50Mbps 带宽   |

### 2.2 安装步骤（以Ubuntu服务端为例）
```bash
# 步骤1：添加软件源
echo "deb https://apt.edgelogguard.com stable main" | sudo tee /etc/apt/sources.list.d/edgelog.list

# 步骤2：安装主程序
sudo apt update && sudo apt install edgelogguard-server

# 步骤3：初始化配置
sudo elg setup --preset=small-team
```

---

## **3. 功能使用指南**
### 3.1 日志管理
#### **实时监控**
!（未完成）[实时监控界面](/assets/realtime.gif)
1. 左侧菜单选择「日志中心」→「实时流」
2. 点击右下角「过滤」按钮设置条件（如 `status >= 400`）
3. 支持高亮显示关键词（右键日志内容→「标记关键词」）

#### **历史查询**
```bash
# 命令行查询示例（支持类SQL语法）
elg query --from="2023-07-01" --to="now-1d" --filter="source='nginx' AND duration_ms > 1000"
```

### 3.2 告警配置
**创建自定义规则**：
1. 导航至「安全中心」→「规则引擎」→「新建规则」
2. 选择触发条件模板：
   ```yaml
   # 示例：频繁登录失败检测
   trigger:
     type: threshold
     field: event_type
     condition: equals("auth_fail")
     window: 5min
     count: 10
   actions:
     - type: email
       receivers: admin@example.com
   ```

### 3.3 可视化仪表盘
**构建自定义视图**：
```json
// 示例：HTTP状态码分布饼图
{
  "type": "pie",
  "query": "SELECT count() FROM logs WHERE source='web' GROUP BY status",
  "display": {
    "title": "Web请求状态分布",
    "colors": {
      "200": "#4CAF50",
      "404": "#FFC107",
      "500": "#F44336"
    }
  }
}
```

---

## **4. 安全配置**
### 4.1 权限控制
**角色权限对照表**：
| 权限项           | 管理员 | 运维人员 | 审计员 | 普通用户 |
|------------------|---------|----------|--------|----------|
| 添加日志源       | ✓       | ✓        | ✗      | ✗        |
| 删除日志         | ✗       | ✗        | ✗      | ✗        |
| 导出审计日志     | ✓       | ✗        | ✓      | ✗        |

**配置多因素认证**：
1. 用户设置页面绑定Google Authenticator
2. 配置策略强制敏感操作需MFA：
   ```bash
   elg policy set --name=mfa_required --rule='action:/(delete|export)/ requires mfa'
   ```

---

## **5. 故障排查**
### 常见问题速查表
| 现象                       | 排查步骤                              | 解决方案                          |
|----------------------------|-------------------------------------|-----------------------------------|
| 日志采集延迟高             | 1. 检查Agent CPU使用率<br>2. 查看网络丢包率 | 限流设置/升级带宽配置              |
| 机器学习检测无结果         | 1. 查看模型加载状态<br>2. 检查特征提取日志 | 重启推理服务/重新训练模型          |
| 无法接收告警通知           | 1. 测试SMTP连接性<br>2. 查看系统队列堆积 | 修正通知配置/清理队列积压          |

---

## **6. 技术支持**
### 获取帮助渠道
- **紧急工单**：SF-TechAdmin@sftech.asia（周一至周五响应时间 ≤3h）（周六日响应时间 ≤1.5h）
- **官方文档**：[点击访问](https://docs.edgelogguard.com)

---

**附录**
- [API参考文档](/api-reference)
- [日志字段字典](/field-glossary)
- [合规性认证](/compliance)

--- 

> **提示**：本手册在线版将随版本持续更新，建议访问官方网站获取最新内容。
