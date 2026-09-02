# Solana Paid Counter

独立的 Solana devnet Counter：每个钱包拥有一个计数器 PDA，初始化、加一和减一每次向合约金库 PDA 支付 `0.001 SOL`。第一个成功调用 `set_owner` 的钱包成为不可变 owner，只有 owner 可以提取金库中超过租金豁免储备的费用。

## 已部署资源

- Program ID：`cnnYUKJ22WztyAumbtrdmrQTW49jPtpWQA6dFnTTa13`
- Owner：`7VioegeemsSaG1SgPxe6Ab8UDi9UcyRzPS5RGaje31oS`
- Config PDA：`7cjocRq2uaqPdiFcWDuv8qz9hH7oVMBLnwJGmfrpcTG8`
- Vault PDA：`H9Wyjuzg95fjHNqZx23F3kj1bCA7FGHZWJ3Cx8H9x5hz`
- Cluster：Solana devnet

## 本地验证

```bash
npm install
npm run test
npm run typecheck
npm run build
NO_DNA=1 cargo build-sbf --manifest-path programs/paid-counter/Cargo.toml
```

前端使用 Wallet Standard 发现钱包，并在请求签名前模拟每笔交易。可通过 `NEXT_PUBLIC_SOLANA_RPC_URL` 与 `NEXT_PUBLIC_COUNTER_PROGRAM_ID` 覆盖默认 devnet 配置。
