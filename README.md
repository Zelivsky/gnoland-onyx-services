# Gnoland Onyx Services

Public infrastructure and tools for [Gnoland Onyx](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/VALIDATOR.md) testnet operators by [Apollo Validator](https://apollo-validator.eu).

[![Onyx](https://img.shields.io/badge/Chain-onyx--1-blue)](https://onyx.testnets.gno.land)
[![RPC](https://img.shields.io/badge/RPC-rpc.apollo--validator.eu-green)](https://rpc.apollo-validator.eu/gnoland-onyx/)
[![Snapshots](https://img.shields.io/badge/Snapshots-snapshots.apollo--validator.eu-orange)](https://snapshots.apollo-validator.eu/gnoland-onyx/)

## Services

| # | Service | Endpoint | Status |
|---|---------|----------|--------|
| 1 | [Public RPC](#public-rpc) | [rpc.apollo-validator.eu/gnoland-onyx/](https://rpc.apollo-validator.eu/gnoland-onyx/) | 🟢 Active |
| 2 | [Snapshots](#snapshots) | [snapshots.apollo-validator.eu/gnoland-onyx/](https://snapshots.apollo-validator.eu/gnoland-onyx/) | 🟢 Active |
| 3 | [Peers List](#peers-list) | [peers.md](peers.md) (auto-updated every 6h) | 🟢 Active |
| 4 | [Installation Guide](#installation-guide) | [guide.md](guide.md) | 🟢 Active |
| 5 | [Validator Monitor Bot](#validator-monitor-bot) | [@gnoland_monitor_apollo_bot](https://t.me/gnoland_monitor_apollo_bot) | 🟢 Active |

---

## Public RPC

Base URL: `https://rpc.apollo-validator.eu/gnoland-onyx/`

### Available Endpoints

| Endpoint | Description |
|---|---|
| `/status` | Node sync status, block height |
| `/net_info` | Network peers info |
| `/validators` | Active validator set |
| `/block?height=N` | Block by height |
| `/commit?height=N` | Block commit |

### Usage Examples

```bash
# Check node status
curl -s https://rpc.apollo-validator.eu/gnoland-onyx/status | jq

# Get current block height
curl -s https://rpc.apollo-validator.eu/gnoland-onyx/status | jq -r '.result.sync_info.latest_block_height'

# Get network peers
curl -s https://rpc.apollo-validator.eu/gnoland-onyx/net_info | jq -r '.result.n_peers'
```

### Configuration for Wallets/Tools

```
RPC URL: https://rpc.apollo-validator.eu/gnoland-onyx/
Chain ID: onyx-1
```

---

## Snapshots

Page: [https://snapshots.apollo-validator.eu/gnoland-onyx/](https://snapshots.apollo-validator.eu/gnoland-onyx/)

Snapshots are created every 6 hours and stored for fast node synchronization. Only the latest snapshot is kept.

See the live snapshot page for the most up-to-date information:
https://snapshots.apollo-validator.eu/gnoland-onyx/

### Download Latest

```bash
# Get snapshot info
curl -s https://snapshots.apollo-validator.eu/api/gnoland-onyx/snapshots/latest | jq

# Download latest snapshot
wget -O gnoland-onyx-snapshot.tar.lz4 https://snapshots.apollo-validator.eu/gnoland-onyx/snapshots/latest.tar.lz4
```

### Restore from Snapshot

```bash
# Install lz4
sudo apt install lz4 -y

# Stop node
sudo systemctl stop gnoland-onyx

# Backup validator state (validators only)
cp ~/gno-onyx/gnoland-data/secrets/priv_validator_state.json ~/priv_validator_state.json.bak

# Download latest snapshot
cd ~/gno-onyx/gnoland-data
wget -O latest.tar.lz4 https://snapshots.apollo-validator.eu/gnoland-onyx/snapshots/latest.tar.lz4

# Verify checksum
sha256sum -c latest.tar.lz4.sha256

# Restore
rm -rf db wal
lz4 -d latest.tar.lz4 | tar -xvf -
rm latest.tar.lz4

# Restore validator state
cp ~/priv_validator_state.json.bak ~/gno-onyx/gnoland-data/secrets/priv_validator_state.json

# Start node
sudo systemctl start gnoland-onyx
```

---

## Peers List

Auto-updated every 6 hours from `/net_info` RPC with TCP verification.

Full list: [peers.md](peers.md) · HTML: [peers.html](https://snapshots.apollo-validator.eu/gnoland-onyx/peers.html)

### Quick Setup

<!-- PEERS:BEGIN -->
Auto-updated every 6 hours · last run: 2026-10-31 06:09 UTC

```bash
PEERS="g1kwcns3j40m6ktw3ewhwlv4jk6dq8zrl33qx37v@116.202.156.139:26756,g1jytmv9tel7esqju93xrxxq6xe8qwjvv72ngacd@152.53.48.29:27656,g10869qdcxxsrsjqr2w4lzkutgamj9l5sr4la4rg@95.217.73.212:54656,g1enctk225g4d8v0gcqtmx4cl6tdham47p5gdyw0@88.198.46.55:38656,g1hk80m9u224ts3xtd0l585hh07hv8rszh0flm23@134.119.194.139:26756,g19jzk586xcle5htkndp9u0kx4hg5splw0069s6u@65.21.15.43:17656,g1d0h0yuytu3c5dq2rp6hck8aeeuclaux0j3zuvd@44.196.164.168:26656,g13jnv6s5kg98ucytee0yyrcpgzkfqfsr5washk2@37.27.127.145:54656,g15uh4xpkdhdc94gakkke2fp9svf7k4dqcf75jcl@95.111.238.100:26656,g10y99z8kfc9cez8qstf4qujr8h7856cu3ugmln3@0.0.0.0:26656,g1grq5zswt0dlwwe7clr4359w70k2ewgse0gcwck@34.247.2.14:26656,g1hfxuds42y4w6k4uv8n2czznyalx433qzgzrnnq@65.108.79.119:54656,g1whafz2mw6zh3g6h4phvtny9gm2yy4rwmg9h7sc@88.198.67.189:29656,g1w44kfhv9a4ruqx70jxqmykychre3ffpnfje8h7@159.195.22.175:61656,g1xqaljl3pty4sjath39ddq7uulsfsm977e26xsq@198.244.200.116:27656,g1wvqps32xgm72gwqrwklhkfu4y8fdst27tackn2@141.98.198.108:26656,g1h99l0phqu3wus5nq8qv778x0frxa6r4n2anl8z@176.9.3.249:28656,g1wuqedh9wzrtkl04jyjhlg3vz60jndnyh0xngzc@168.119.73.186:26656,g1trep88l2yex6rlklzw52zf6hhj7dcgfr4jw06h@0.0.0.0:26656,g1zjxshnsanpkn5en0ajy35upk98mctyjt6xs3yr@0.0.0.0:26656,g1fkenv95pqswdm83fereter3tr384cfdkyhaxrh@65.108.128.168:26656,g18fky8kfslt69c57mysdtw6ce823wz797t2krwz@162.250.190.214:26656,g1ax8w67f07gq7kfhx48akhj820n9l0uumwnqy3k@65.108.198.182:26656,g1r6jgch2ax4kvqrtju8v3pgk5pmwe9myjle9uws@65.109.104.118:36656,g19q6s7zk4u0pnazxzdz4sl3c3wavlhh048g27p0@135.181.78.21:37656,g1adeecqwltcvenkj842lt335d455kny92hvkhpy@0.0.0.0:26656,g17537c8pzk3y4gtam8jvk4tf8jcavy80n2zax5w@65.109.30.26:54656,g1ecf7np0kwy264agt4a7d37rhzqn94sztzkygfj@46.232.248.39:54656,g1fzle2wra6y2c6m3mn9u0xc534ygu37sjtf8naw@171.246.188.30:26668,g1vp8jzr2cr70xtcxlretm4xutvwujlpdsm89zrv@149.50.96.58:41696,g10jrjsyzgm4p0hy6ewd5lkt9ztsx48lqpy6zk0q@0.0.0.0:28656,g10s3cw8kv05zx338tkcsx438sm5p5dwmfa6jgy4@65.109.124.135:54656,g14vxnkg708eguxdejukk5emq6zsmwusda8a8lle@34.248.195.29:26656,g17t7vlg7hjvsldqj06zenkxpktzntkp62thg9a7@65.109.106.214:55686,g17pujqhhnkxknjas3fteuxq6l7w8kzueq0eahkd@148.72.141.245:26672,g10y5062yujmr7wjf2eqqrnfzpv8c52x6epf3yf5@185.16.39.172:59656,g1ackmzg8myw06lfne6djqlmdw7prt477y0z5erk@209.209.8.93:46656,g1dqqmysnfv38adpm8vhaurc7w48gz00y6hm65nd@0.0.0.0:26656,g1qkvmqn7wp6ch76zg5ady7ppgngjylhd7uucp87@0.0.0.0:26656,g1f0rqz8h3cf4gsq78cnncsfwac2ssf4cak4n95n@207.180.207.177:26656,g1z6npzw8p27gklxzzccsx4gq6kxkc8ve5xwatxu@52.57.210.177:26656,g1wfu406mz57cv2cq68agrkefd4g6pw0ew4kyvlu@65.109.50.163:54656,g1raankm9dxx4xm0g20666ft4x0qr09vgwlryfp2@117.1.193.44:26656,g1x5mlj5ava0dw9vkf4j6admjlzswm6f06p44krn@100.24.196.77:26656,g1r93ua5fzez4gp8xpdxnxghll9supkye54k5tgd@167.86.76.130:6656,g1vcua8y0pycjmcg5fvu2hh3azen5nf0p2smrqzs@135.181.216.107:26656,g1hryyznzq2msp89le5qw5javs7ge0vn0xc6am3v@34.203.181.163:26656,g1e3pftctakyx58mzapqs4syc4h0jcxwtnraq00c@185.248.24.16:47656,g1qss40s3c30f02z7vu52welzxmugwgrh5nzhnp2@0.0.0.0:26656"

cd ~/gno-onyx/gnoland-data
gnoland config set p2p.persistent_peers "$PEERS" --config-path config/config.toml

sudo systemctl restart gnoland-onyx
```
<!-- PEERS:END -->

### Official Seeds

```
g1x5mlj5ava0dw9vkf4j6admjlzswm6f06p44krn@seed-1.onyx.testnets.gno.land:26656
g1grq5zswt0dlwwe7clr4359w70k2ewgse0gcwck@seed-2.onyx.testnets.gno.land:26656
```

---

## Installation Guide

Full guide: [guide.md](guide.md)

### Quick Start

```bash
# Download binary (pin version from UPGRADES.md — currently v1.6.0-rc.1)
mkdir -p ~/go-onyx/bin
wget -O ~/go-onyx/bin/gnoland \
  https://github.com/gnolang/gno/releases/download/v1.6.0-rc.1/gnoland_linux_amd64
chmod +x ~/go-onyx/bin/gnoland
~/go-onyx/bin/gnoland version

# GNOROOT — checkout of the same tag
git clone https://github.com/gnolang/gno.git ~/gno-onyx-src
cd ~/gno-onyx-src && git checkout v1.6.0-rc.1

# Download genesis
mkdir -p ~/gno-onyx/gnoland-data
wget -O ~/gno-onyx/genesis.json https://github.com/gnolang/gno/releases/download/chain/onyx/genesis.json
sha256sum ~/gno-onyx/genesis.json
# Expected: 4b006fd7ccdec052865accc84dd29b2b76f8b57b2560789a15eedaa88f0e26c5

# Initialize and configure
export GNOROOT=$HOME/gno-onyx-src
gnoland config init --config-path ~/gno-onyx/gnoland-data/config/config.toml
gnoland secrets init --data-dir ~/gno-onyx/gnoland-data/secrets

# Configure ports (47xxx to avoid conflicts)
CFG=~/gno-onyx/gnoland-data/config/config.toml
gnoland config set rpc.laddr "tcp://0.0.0.0:47657" --config-path $CFG
gnoland config set p2p.laddr "tcp://0.0.0.0:47656" --config-path $CFG
gnoland config set proxy_app "tcp://127.0.0.1:47658" --config-path $CFG
gnoland config set p2p.persistent_peers "g1x5mlj5ava0dw9vkf4j6admjlzswm6f06p44krn@seed-1.onyx.testnets.gno.land:26656,g1grq5zswt0dlwwe7clr4359w70k2ewgse0gcwck@seed-2.onyx.testnets.gno.land:26656" --config-path $CFG

# Required settings
gnoland config set application.prune_strategy syncable --config-path $CFG
gnoland config set consensus.timeout_commit 3s --config-path $CFG
gnoland config set consensus.peer_gossip_sleep_duration 10ms --config-path $CFG
gnoland config set p2p.flush_throttle_timeout 10ms --config-path $CFG

# Start node
gnoland start --chainid onyx-1 --genesis ~/gno-onyx/genesis.json \
  --skip-genesis-sig-verification --data-dir ~/gno-onyx/gnoland-data
```

---

## Validator Monitor Bot

Telegram bot monitoring the Gnoland Onyx testnet — and Gnoland mainnet from the same bot: node health, missed-block alerts, peers and snapshots.

**Bot:** [@gnoland_monitor_apollo_bot](https://t.me/gnoland_monitor_apollo_bot)

Missed-block alerts are **watchlist-based**: you get an alert only for validators
you add with `/add` — on every network where that validator is active.

**Commands:**
| Command | Description |
|---------|-------------|
| `/status` | Status of both networks (mainnet + Onyx) |
| `/peers` | Live peers + verified list timestamp |
| `/snapshot` | Latest snapshots of both networks |
| `/add <moniker or address>` | Watch a validator on every network where active |
| `/remove <moniker or address>` | Remove from watchlist |
| `/list` | Your watchlist with active-set status |
| `/check` | Uptime check for your watchlist |
| `/map <moniker or address>` | Validator info lookup |
| `/threshold <n>` | Missed-blocks alert threshold |

---

## Chain Information

| Parameter | Value |
|---|---|
| Chain ID | onyx-1 |
| Binary | v1.6.0-rc.1 (pin from UPGRADES.md) |
| Genesis | [Download](https://github.com/gnolang/gno/releases/download/chain/onyx/genesis.json) |
| Genesis SHA256 | `4b006fd7ccdec052865accc84dd29b2b76f8b57b2560789a15eedaa88f0e26c5` |
| P2P Port | 47656 |
| RPC Port | 47657 |
| ABCI Port | 47658 |
| Start flag | `--skip-genesis-sig-verification` (required) |

---

## Links

- [VALIDATOR.md](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/VALIDATOR.md)
- [UPGRADES.md](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/UPGRADES.md)
- [Faucet](https://onyx.testnets.gno.land/faucet)
- [Validators](https://onyx.testnets.gno.land/r/gnops/valopers)
- [GnoScan](https://gnoscan.io)

---

*Provided by [Apollo Validator](https://apollo-validator.eu)*
