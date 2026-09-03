# Solana Paid Counter 需求说明书

## 1. 项目目标

提供一个部署在 Solana devnet 的付费计数器。每个钱包拥有独立的链上计数器；用户初始化、递增或递减计数器时，均向程序金库支付固定费用。程序设置唯一且不可变的 owner，只有 owner 可以提取累计费用。

本文档是项目后续开发、验收和回归测试的需求基线；实现细节如与本文档冲突，应先确认并更新需求。

## 2. 功能需求

### 2.1 Owner 与金库

- 第一个成功执行 `set_owner` 的钱包成为全局 owner。
- Owner 一经设置不可更改，重复设置必须失败。
- 设置 owner 时同时初始化全局 Config PDA 与 Vault PDA。
- 只有 owner 可以从 Vault PDA 提取费用。
- 提取后必须保留 Vault PDA 的租金豁免余额。
- 提取金额为 `0` 时表示提取全部可用费用；余额不足或可用费用为零时必须失败。

### 2.2 钱包计数器

- 每个钱包按其公钥派生唯一 Counter PDA，钱包之间的数据互不影响。
- Counter 初始值为 `0`，数值类型为有符号 64 位整数。
- 已初始化的 Counter 支持每次加一或减一。
- 只有 Counter 中记录的 authority 可以修改该 Counter。
- 数值溢出或下溢时操作必须失败。

### 2.3 收费

- 初始化 Counter、加一、减一每次固定收取 `0.001 SOL`（`1,000,000` lamports）。
- 费用从操作钱包转入全局 Vault PDA。
- 设置 owner 和提取费用不收取上述业务操作费；用户仍承担 Solana 网络交易费及账户初始化所需租金。

### 2.4 Dapp

- 使用 Wallet Standard 发现、连接、切换和断开支持 Solana 交易的钱包。
- 仅面向 Solana devnet，并清晰显示当前网络。
- 连接钱包后读取其 Counter、全局 owner 和 Vault 可提取费用。
- 未设置 owner 时允许当前钱包发起一次性 owner 设置。
- 未初始化 Counter 时，主要操作应创建 Counter；初始化后提供加一和减一。
- 当前钱包为 owner 时显示提取全部可用费用操作。
- 每笔交易请求签名前必须先模拟；模拟失败不得请求签名。
- 展示交易状态、错误信息和已确认交易的 Solana Explorer 链接。
- 默认 RPC 与 Program ID 可分别通过 `NEXT_PUBLIC_SOLANA_RPC_URL` 和 `NEXT_PUBLIC_COUNTER_PROGRAM_ID` 覆盖。

## 3. 链上接口与数据约束

### 指令

| Tag | 指令 | 数据 | 账户顺序 |
| --- | --- | --- | --- |
| `0` | 初始化 Counter | 无 | authority（签名、可写）、counter（可写）、vault（可写）、system program |
| `1` | 加一 | 无 | authority（签名、可写）、counter（可写）、vault（可写）、system program |
| `2` | 减一 | 无 | authority（签名、可写）、counter（可写）、vault（可写）、system program |
| `3` | 设置 owner | 无 | candidate（签名、可写）、config（可写）、vault（可写）、system program |
| `4` | 提取费用 | `u64` 小端金额 | owner（签名、可写）、config、vault（可写）、destination（可写） |

### PDA

- Counter PDA：seeds 为 `["counter", authority]`。
- Config PDA：seeds 为 `["config"]`。
- Vault PDA：seeds 为 `["vault"]`。

### 账户布局（版本 1）

- Counter：43 bytes；discriminator、version、`i64` value、authority、bump。
- Config：35 bytes；discriminator、version、owner、bump。
- Vault：2 bytes；discriminator、bump。
- 程序必须校验账户 owner、长度、discriminator、版本、PDA seeds、签名与可写权限。

## 4. 非功能与安全要求

- Solana 程序使用 Pinocchio，保持 `no_std` 与无 allocator 运行方式。
- 所有 lamports 和计数运算必须使用受检算术。
- 不得允许非 owner 提款、非 authority 修改计数器、伪造 PDA 或将 Vault 作为提款目标。
- 前端以 `confirmed` commitment 读取、模拟和确认交易。
- 不在源码中保存私钥、助记词或其他凭据。
- UI 应提供清晰的加载、禁用、成功、失败与钱包拒签反馈，并具备基本键盘和读屏可访问性。

## 5. 验收标准

- 单元测试覆盖 PDA 派生、指令 tag、账户顺序与权限、固定费用，以及 withdraw-all 的 `0` 编码。
- `npm run test`、`npm run typecheck`、`npm run lint`、`npm run build` 均通过。
- `NO_DNA=1 cargo build-sbf --manifest-path programs/paid-counter/Cargo.toml` 构建成功。
- Devnet 上可完成 owner 设置、Counter 初始化、加一、减一与 owner 提款的完整流程。
- 生产 Dapp 与独立 GitHub 仓库保持关联，README 记录当前 Dapp、Program ID、owner 及 PDA 地址。

## 6. 当前部署基线

当前部署地址与链上标识以 `README.md` 的“已部署资源”为准。变更部署或迁移程序时，必须同步更新 README，并重新验证本说明书中的完整流程。
