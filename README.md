# StandX 做市策略机器人

## 简介

这是一个与 StandX 交互的做市策略脚本，旨在通过智能挂单策略尽量无损获取挂单积分以及提升 maker uptime（挂单时长）。

使用过程有任何问题可以加群交流：https://t.me/bitcat365/4979

## 核心特性

1. **实时订单簿监控**：使用 WebSocket 获取实时订单簿数据，计算加权中间价，相比 Symbol Price 方式时效性更强，对价格更敏感。

2. **ATR 波动率风控**：从币安现货获取 K 线价格，计算 ATR（Average True Range）波动率指标，避免在高波动时期被吃单。

3. **市场领先响应**：从币安获取订单簿价格推送，得到市场领先的价格响应，计算超短时内的价格变动，做到提前风控撤单，降低被吃单风险。

4. **智能仓位管理**：支持自动修复仓位、自动平仓等多种策略模式，灵活应对市场变化。

5. **支持多种运行方式**：同时支持使用API token以及使用以太坊钱包keystore文件的方式。建议使用API token。

## 风险提示

尽管此脚本已经努力降低吃单的可能性，但是在市场高波动情况下，10bps（0.1%）的狭窄区间，依然存在被吃单的可能。请仔细权衡自己的风险承受能力后再使用此脚本。

## 快速开始

### 前置要求

- Python 3.8+
- 可访问币安 API 的网络环境
- StandX 账户及 API 权限

### 1. 创建虚拟环境（可选，但推荐）

建议使用 Python 虚拟环境来运行此项目，以避免依赖冲突。

```bash
# 创建虚拟环境
python -m venv myvenv

# 激活虚拟环境
# Windows
myvenv\Scripts\activate

# macOS/Linux
source myvenv/bin/activate
```

### 2. 安装依赖

在激活虚拟环境后，安装项目依赖：

```bash
pip install -r requirements.txt
```

### 3. 配置环境变量

首次运行时，复制 `.env.example` 文件为 `.env`，并根据您的需求修改配置：

```bash
cp .env.example .env
```

然后编辑 `.env` 文件，配置相关参数。

#### API Token 配置说明

通过访问 https://standx.com/user/session 生成并获取 API Token 以及 ED25519 私钥，然后填写在配置文件中。

### 4. 生成 Keystore 文件（可选）

官方最初支持使用以太坊钱包私钥签名的交易方式，本项目依然支持此种方式，但是为确保不使用明文私钥，项目支持使用 keystore 的运行模式。注意：使用此方法时，每次启动需手动输入密码，因此无法使用守护方式（如 systemd）运行程序。

有以下两种方式生成 keystore 文件：

- 生成新地址
- 导入现有私钥

运行以下命令生成 keystore 文件：

```bash
python eth_keystore.py
```

按照提示选择生成新地址或导入私钥，并完成相关操作。

### 5. 运行做市脚本

最后，运行以下命令启动做市机器人：

```bash
python only_maker.py
```

## 注意事项

- **配置检查**：请确保在运行做市机器人之前，已正确配置 `.env` 文件和 keystore 文件。
- **资金充足**：做市机器人会根据配置的策略自动挂单和撤单，请确保您的账户有足够的资金和权限。
- **测试先行**：建议在测试环境中先进行测试，确保一切正常后再在生产环境中运行。
- **避免冲突**：建议做市账号不要在客户端进行其他手动操作，以避免相互影响。
- **仓位修复**：建议开启 `STANDX_MAKER_FIX_ORDER_ENABLED`，做到被吃单后修复仓位。
- **自动平仓**：如需自动平仓模式（挂单成交后立即市价平仓，避免持仓风险），可开启 `STANDX_MAKER_AUTO_CLOSE_POSITION=true`。默认关闭，不影响原有做市逻辑。

## 钉钉通知配置（可选）

支持在订单成交（被吃单）时推送钉钉机器人通知，包含地址和交易信息。

### 配置方法

在 `.env` 文件中添加：

```bash
DINGTALK_WEBHOOK='https://oapi.dingtalk.com/robot/send?access_token=xxx'  # 完整的 Webhook 地址
DINGTALK_KEYWORD='Standx'  # 钉钉机器人安全设置中的关键词
```

### 功能说明

- **默认开启**：配置了 `DINGTALK_WEBHOOK` 后自动启用
- **关闭通知**：将 `DINGTALK_WEBHOOK` 留空即可关闭通知
- **关键词匹配**：`DINGTALK_KEYWORD` 必须与钉钉机器人安全设置中的关键词一致，默认为 `Standx`
- **通知内容**：交易对、买卖方向、成交价格、成交数量、当前仓位、钱包地址

### 获取钉钉机器人 Webhook

1. 在钉钉群中添加自定义机器人
2. 安全设置选择"自定义关键词"，添加关键词（需与 `DINGTALK_KEYWORD` 一致）
3. 复制完整的 Webhook 地址到 `DINGTALK_WEBHOOK` 配置项

### 日志查看

程序运行日志保存在 `logs/` 目录下：
- `only_maker.log`：主程序运行日志
- `error.log`：错误日志

### 调试模式

如需更详细的日志信息，可在 `.env` 文件中设置：
```bash
LOG_LEVEL='DEBUG'
```
---

**免责声明**：本软件仅供学习和研究使用，作者不对因使用本软件而产生的任何直接或间接损失负责。使用前请确保了解相关风险。