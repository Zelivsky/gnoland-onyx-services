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

```bash
# Copy the persistent_peers line from peers.md
# Edit ~/gno-onyx/gnoland-data/config/config.toml
persistent_peers = "<peers_from_peers.md>"

# Restart node
sudo systemctl restart gnoland-onyx
```

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
# Download binary (pin version from UPGRADES.md — currently v1.5.0)
mkdir -p ~/go-onyx/bin
wget -O ~/go-onyx/bin/gnoland \
  https://github.com/gnolang/gno/releases/download/v1.5.0/gnoland_linux_amd64
chmod +x ~/go-onyx/bin/gnoland
~/go-onyx/bin/gnoland version

# GNOROOT — checkout of the same tag
git clone https://github.com/gnolang/gno.git ~/gno-onyx-src
cd ~/gno-onyx-src && git checkout v1.5.0

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

## Chain Information

| Parameter | Value |
|---|---|
| Chain ID | onyx-1 |
| Binary | v1.5.0 (pin from UPGRADES.md) |
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
