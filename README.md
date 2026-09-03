# Solana Paid Counter

独立的 Solana devnet Counter：每个钱包拥有一个计数器 PDA，初始化、加一和减一每次向合约金库 PDA 支付 `0.001 SOL`。第一个成功调用 `set_owner` 的钱包成为不可变 owner，只有 owner 可以提取金库中超过租金豁免储备的费用。

## 已部署资源

- Dapp：https://solana-paid-counter-39.vercel.app
- Program ID：`cFf5Vzhasx99wtJ3x9ivnGYYfoinwRtN6JjxyvtNJr9`
- Owner：尚未设置（首次 `set_owner` 留给 Playground 调用）
- Config PDA：`FYPCqnVydB2Yie4dWNQ1vHZKvRwqENsxHmmmvBeNj6nd`（尚未创建）
- Vault PDA：`7CpzeBYMSdeHGWtEaU5cgxFp9HpZuXsjfdNUzfiNmSGK`（尚未创建）
- Cluster：Solana devnet
- 部署交易：`2Q6gAfGhKYos6r2RfRWmdSmV1y6o2GrTgFQCiYVR268U3ENWAgPt9ZPRZ9pmcFPRh8MsLg8jrf9jQtM4sDcRfkxz`

完整接口见 `docs/ABI.md`，操作、验证与排错见 `docs/项目使用说明书.md`。

## 本地验证

```bash
npm install
npm run test
npm run typecheck
npm run build
NO_DNA=1 cargo build-sbf --manifest-path programs/paid-counter/Cargo.toml
```

前端使用 Wallet Standard 发现钱包，并在请求签名前模拟每笔交易。可通过 `NEXT_PUBLIC_SOLANA_RPC_URL` 与 `NEXT_PUBLIC_COUNTER_PROGRAM_ID` 覆盖默认 devnet 配置。
