# Gnoland Onyx — Installation Guide

## Overview

This guide covers setting up a Gnoland Onyx testnet node and registering as a validator candidate.

**Network:** Onyx (onyx-1)
**Branch:** mainnet code, pinned to release tag (e.g. v1.5.0)
**Ports:** 47xxx (P2P: 47656, RPC: 47657, ABCI: 47658)

## Prerequisites

- Ubuntu 22.04+ or similar Linux
- Go 1.22+
- 4GB+ RAM
- 50GB+ disk

## 1. Install Go

```bash
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
echo 'export PATH=/usr/local/go/bin:$HOME/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

## 2. Download Binary and GNOROOT

Onyx runs mainnet code one release candidate ahead. Pin the exact version
from the last row of `misc/deployments/onyx.gno.land/UPGRADES.md` (currently **v1.5.0**).
Never build from master — use the release tag.

```bash
# Prebuilt binary
mkdir -p ~/go-onyx/bin
wget -O ~/go-onyx/bin/gnoland \
  https://github.com/gnolang/gno/releases/download/v1.5.0/gnoland_linux_amd64
chmod +x ~/go-onyx/bin/gnoland

# Verify
~/go-onyx/bin/gnoland version
# gnoland version: v1.5.0

# GNOROOT — checkout of the same tag (node reads gnovm/stdlibs from it)
git clone https://github.com/gnolang/gno.git ~/gno-onyx-src
cd ~/gno-onyx-src && git checkout v1.5.0
```

## 3. Download Genesis

```bash
mkdir -p ~/gno-onyx/gnoland-data
wget -O ~/gno-onyx/genesis.json \
  https://github.com/gnolang/gno/releases/download/chain/onyx/genesis.json

# Verify checksum
sha256sum ~/gno-onyx/genesis.json
# Expected: 4b006fd7ccdec052865accc84dd29b2b76f8b57b2560789a15eedaa88f0e26c5
```

## 4. Initialize Node

```bash
export GNOROOT=$HOME/gno-onyx-src

gnoland config init --config-path ~/gno-onyx/gnoland-data/config/config.toml
gnoland secrets init --data-dir ~/gno-onyx/gnoland-data/secrets
```

## 5. Configure Node

```bash
CFG=~/gno-onyx/gnoland-data/config/config.toml

# Ports (to avoid conflicts with other networks)
gnoland config set rpc.laddr "tcp://0.0.0.0:47657" --config-path $CFG
gnoland config set p2p.laddr "tcp://0.0.0.0:47656" --config-path $CFG
gnoland config set proxy_app "tcp://127.0.0.1:47658" --config-path $CFG

# Persistent peers
gnoland config set p2p.persistent_peers "g1x5mlj5ava0dw9vkf4j6admjlzswm6f06p44krn@seed-1.onyx.testnets.gno.land:26656,g1grq5zswt0dlwwe7clr4359w70k2ewgse0gcwck@seed-2.onyx.testnets.gno.land:26656" --config-path $CFG

# Required settings
gnoland config set application.prune_strategy syncable --config-path $CFG
gnoland config set consensus.timeout_commit 3s --config-path $CFG
gnoland config set consensus.peer_gossip_sleep_duration 10ms --config-path $CFG
gnoland config set p2p.flush_throttle_timeout 10ms --config-path $CFG

# Recommended
gnoland config set mempool.size 10000 --config-path $CFG
gnoland config set p2p.max_num_outbound_peers 40 --config-path $CFG
gnoland config set p2p.pex true --config-path $CFG
gnoland config set moniker "YourMoniker" --config-path $CFG
gnoland config set p2p.external_address "YOUR_IP:47656" --config-path $CFG
```

## 6. Create Systemd Service

```bash
sudo tee /etc/systemd/system/gnoland-onyx.service > /dev/null << 'EOF'
[Unit]
Description=Gnoland Onyx Node
After=network-online.target
Wants=network-online.target

[Service]
User=YOUR_USER
WorkingDirectory=/home/YOUR_USER/gno-onyx-src
Environment=GNOROOT=/home/YOUR_USER/gno-onyx-src
Environment=GOROOT=/usr/local/go
Environment=PATH=/usr/local/go/bin:/home/YOUR_USER/go-onyx/bin:/usr/bin:/bin
Environment=HOME=/home/YOUR_USER
ExecStart=/home/YOUR_USER/go-onyx/bin/gnoland start --chainid onyx-1 --genesis /home/YOUR_USER/gno-onyx/genesis.json --skip-genesis-sig-verification --data-dir /home/YOUR_USER/gno-onyx/gnoland-data --log-level info
Restart=on-failure
RestartSec=5s
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=gnoland-onyx

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable gnoland-onyx
sudo systemctl start gnoland-onyx
```

> **Note:** `--skip-genesis-sig-verification` is required — Onyx genesis has
> placeholder/invalidated signatures.

## 7. Sync from Snapshot (Fast)

```bash
# Install lz4
sudo apt install lz4 -y

# Stop node
sudo systemctl stop gnoland-onyx

# Download and extract snapshot
cd ~/gno-onyx/gnoland-data
wget -O latest.tar.lz4 https://snapshots.apollo-validator.eu/gnoland-onyx/snapshots/latest.tar.lz4
rm -rf db wal
lz4 -d latest.tar.lz4 | tar -xvf -
rm latest.tar.lz4

# Start node
sudo systemctl start gnoland-onyx
```

## 8. Check Sync Status

```bash
# Check height
curl -s http://127.0.0.1:47657/status | jq -r .result.sync_info.latest_block_height

# Check if catching up
curl -s http://127.0.0.1:47657/status | jq -r .result.sync_info.catching_up

# View logs
sudo journalctl -u gnoland-onyx -f
```

## 9. Register as Validator

Register only after the node is **fully synced** (`catching_up: false`).

### Get validator public key
```bash
export GNOROOT=$HOME/gno-onyx-src
gnoland secrets get validator_key --data-dir ~/gno-onyx/gnoland-data/secrets
# Note the gpub1... value
```

### Get testnet GNOT
Visit https://onyx.testnets.gno.land/faucet and request tokens for your g1... address.

### Register
```bash
gnokey maketx call \
  --pkgpath gno.land/r/gnops/valopers \
  --func Register \
  --args "YourMoniker" \
  --args "Your description" \
  --args "data-center" \
  --args "YOUR_G1_ADDRESS" \
  --args "YOUR_GPUB1_KEY" \
  --gas-fee 1000000ugnot --gas-wanted 50000000 \
  --chainid onyx-1 \
  --remote tcp://127.0.0.1:47657 \
  --broadcast \
  YOUR_KEY_NAME
```

### Wait for GovDAO approval
After registration you become a **candidate**. A GovDAO member must create and pass a proposal to add you to the active validator set.

Check status:
- Registered candidates: https://onyx.testnets.gno.land/r/gnops/valopers
- Active validators: https://onyx.testnets.gno.land/r/sys/validators/v0

## 10. Useful Commands

```bash
# Service management
sudo systemctl status gnoland-onyx
sudo systemctl restart gnoland-onyx
sudo journalctl -u gnoland-onyx -f

# Node info
curl -s http://127.0.0.1:47657/status | jq .result.node_info
curl -s http://127.0.0.1:47657/net_info | jq .result.n_peers

# Validator info
curl -s http://127.0.0.1:47657/validators | jq '.result.validators | length'

# Wallet balance
gnokey query --remote tcp://127.0.0.1:47657 auth/accounts/YOUR_G1_ADDRESS
```

## Upgrading

Onyx is upgraded whenever mainnet is. Check the last row of
`misc/deployments/onyx.gno.land/UPGRADES.md` for the required version, then:

```bash
sudo systemctl stop gnoland-onyx

# Update binary to the new tag
wget -O ~/go-onyx/bin/gnoland \
  https://github.com/gnolang/gno/releases/download/<NEW_TAG>/gnoland_linux_amd64
chmod +x ~/go-onyx/bin/gnoland

# Update GNOROOT checkout
cd ~/gno-onyx-src && git fetch && git checkout <NEW_TAG>

sudo systemctl start gnoland-onyx
```

## Troubleshooting

### Validator not signing blocks
Reset validator state:
```bash
sudo systemctl stop gnoland-onyx
echo '{"height":"0","round":"0","step":0}' > ~/gno-onyx/gnoland-data/secrets/priv_validator_state.json
sudo systemctl start gnoland-onyx
```

**Format note:** `height` and `round` must be strings, `step` must be a number:
- Correct: `{"height":"0","round":"0","step":0}`
- Wrong: `{"height":0,"round":0,"step":0}`

### Node won't start (GNOROOT)
Make sure GNOROOT is set in the systemd service environment and points to
the checkout matching the binary version.

### Port conflicts
Onyx uses ports 47xxx. Make sure no other service uses these ports:
```bash
ss -tlnp | grep 47
```

### Logs suppressed
If logs seem to disappear, check with:
```bash
sudo journalctl -u gnoland-onyx --no-pager -n 1000
```

## Links

| Resource | URL |
|----------|-----|
| VALIDATOR.md | https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/VALIDATOR.md |
| UPGRADES.md | https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/UPGRADES.md |
| Genesis | https://github.com/gnolang/gno/releases/download/chain/onyx/genesis.json |
| Faucet | https://onyx.testnets.gno.land/faucet |
| Validators | https://onyx.testnets.gno.land/r/gnops/valopers |
| Active Set | https://onyx.testnets.gno.land/r/sys/validators/v0 |
| Public RPC | https://rpc.apollo-validator.eu/gnoland-onyx/ |
| Snapshots | https://snapshots.apollo-validator.eu/gnoland-onyx/ |
| GnoScan | https://gnoscan.io |

---

*Provided by Apollo Validator*
