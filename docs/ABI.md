# Paid Counter ABI

本项目使用自有二进制 ABI，不使用或生成 Anchor IDL。所有整数均为小端编码。

## Program 与 PDA

- Program ID：`cFf5Vzhasx99wtJ3x9ivnGYYfoinwRtN6JjxyvtNJr9`
- Counter PDA：`["counter", authority_pubkey]`
- Config PDA：`["config"]`，当前地址 `FYPCqnVydB2Yie4dWNQ1vHZKvRwqENsxHmmmvBeNj6nd`
- Vault PDA：`["vault"]`，当前地址 `7CpzeBYMSdeHGWtEaU5cgxFp9HpZuXsjfdNUzfiNmSGK`

## 指令

| Tag | 名称 | 指令数据 | 账户（严格顺序） |
| --- | --- | --- | --- |
| `0` | `initialize_counter` | `u8 tag` | authority（signer、writable）、counter（writable）、vault（writable）、system program |
| `1` | `increment` | `u8 tag` | authority（signer、writable）、counter（writable）、vault（writable）、system program |
| `2` | `decrement` | `u8 tag` | authority（signer、writable）、counter（writable）、vault（writable）、system program |
| `3` | `set_owner` | `u8 tag` | candidate（signer、writable）、config（writable）、vault（writable）、system program |
| `4` | `withdraw` | `u8 tag` + `u64 amount` | owner（signer、writable）、config（readonly）、vault（writable）、destination（writable） |

`withdraw.amount == 0` 表示提取全部可用费用，同时保留 Vault 的租金豁免余额。初始化、递增、递减固定收费 `1,000,000` lamports。

## 账户布局

- Counter（43 bytes）：`u8 discriminator=1`、`u8 version=1`、`i64 value`、`[u8;32] authority`、`u8 bump`。
- Config（35 bytes）：`u8 discriminator=2`、`u8 version=1`、`[u8;32] owner`、`u8 bump`。
- Vault（2 bytes）：`u8 discriminator=3`、`u8 bump`。

## 错误

程序使用标准 `ProgramError` 表达账户数量、签名、程序 ID、账户数据、重复初始化、PDA、指令数据等错误；自定义错误码为：`0` Unauthorized、`1` Overflow、`2` InsufficientFunds。
