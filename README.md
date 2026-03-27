# Polymarket Wallet On-Chain Activity

On-chain USDC and CTF (Conditional Token Framework) transfer data for 99 active Polymarket wallets on Polygon.

## Data Summary

| Resource | Format | Count | Rows | Size |
|----------|--------|-------|------|------|
| USDC transfers | Parquet (per wallet) | 99 wallets | ~30M | 1.8 GB |
| CTF transfers | Parquet (per wallet) | 129 wallets | ~17M | 979 MB |
| Deep drill | JSON (per wallet) | 8 wallets | — | 100 MB |
| PnL rankings | JSON | — | — | — |
| Strategy analysis | JSON | — | — | — |
| Market makers | JSON | — | — | — |

## Schemas

### USDC Transfers (`usdc_transfers/*.parquet`)

Every USDC transfer (native + PoS-bridged) involving a tracked wallet.

| Column | Type | Description |
|--------|------|-------------|
| `wallet` | string | Tracked wallet address |
| `block_number` | int64 | Polygon block number |
| `timestamp` | string | ISO 8601 timestamp |
| `tx_hash` | string | Transaction hash |
| `from_addr` | string | Sender address |
| `to_addr` | string | Receiver address |
| `usdc_amount` | float64 | USDC amount (6-decimal adjusted) |
| `flow_type` | string | `in` or `out` relative to wallet |
| `from_label` | string | Label for sender |
| `to_label` | string | Label for receiver |
| `usdc_contract` | string | `native` or `pos` |

### CTF Transfers (`ctf_transfers/*.parquet`)

ERC-1155 token transfers for Polymarket outcome tokens.

| Column | Type | Description |
|--------|------|-------------|
| `wallet` | string | Tracked wallet address |
| `block_number` | int64 | Polygon block number |
| `timestamp` | string | ISO 8601 timestamp |
| `tx_hash` | string | Transaction hash |
| `event_type` | string | `trade_in`, `trade_out`, `redeem`, `merge`, `split`, etc. |
| `token_id` | string | ERC-1155 token ID (maps to market outcome) |
| `amount` | float64 | Token amount |
| `direction` | string | `to` (received) or `from` (sent) |
| `from_addr` | string | Sender address |
| `to_addr` | string | Receiver address |

## Querying with DuckDB

```sql
-- Total USDC volume per wallet
SELECT wallet, SUM(usdc_amount) as total_volume
FROM read_parquet('data/2026-03-27/usdc_transfers/*.parquet')
GROUP BY wallet ORDER BY total_volume DESC LIMIT 20;

-- CTF trade activity by event type
SELECT event_type, COUNT(*) as cnt
FROM read_parquet('data/2026-03-27/ctf_transfers/*.parquet')
GROUP BY event_type ORDER BY cnt DESC;

-- Single wallet timeline
SELECT timestamp, flow_type, usdc_amount, from_label, to_label
FROM read_parquet('data/2026-03-27/usdc_transfers/0x006cc834cc092684f1b56626e23bedb3835c16ea.parquet')
ORDER BY timestamp;
```

## Sources

- USDC Transfer events scraped from Polygon via Polygonscan / RPC
- CTF token events from Polymarket exchange contracts on Polygon
- Market resolution data from Polymarket Gamma API

## License

CC-BY-4.0
