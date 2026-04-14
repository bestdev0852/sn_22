# Bittensor (Desearch) Miner Setup Guide

This guide details the process for setting up and running a Bittensor Desearch miner using the Desearch repository.

## Prerequisites
Before starting, ensure you have:

- **PM2:** A process manager to maintain your miner. If not installed, see [PM2 Installation](https://pm2.io/docs/runtime/guide/installation/).

- **Environment Variables:** Set the necessary variables as per the [Environment Variables Guide](./env_variables.md).

## Setup Process

## 1. Clone the desearch repository and install dependencies
Clone and install the Desearch repository in editable mode:

```sh
git clone https://github.com/Desearch-ai/subnet-22.git
cd desearch
python -m pip install -r requirements.txt
python -m pip install -e .
```

### 2. Configure and Run the Miner
Configure and launch the miner using PM2:

```sh
pm2 start neurons/miners/miner.py \
--miner.name desearch \
--interpreter <path-to-python-binary> -- \
--wallet.name <wallet-name> \
--wallet.hotkey <wallet-hotkey> \
--netuid <netuid> \
--subtensor.network <network> \
--axon.port <port>

# Example
pm2 start neurons/miners/miner.py --interpreter /usr/bin/python3 --name miner_1 -- --wallet.name miner --wallet.hotkey default --subtensor.network testnet --netuid 41 --axon.port 14001
```

#### Variable Explanation
- `--wallet.name`: Your wallet's name.
- `--wallet.hotkey`: Your wallet's hotkey.
- `--netuid`: Network UID, `41` for testnet.
- `--subtensor.network`: Choose network (`finney`, `test`, `local`, etc).
- `--logging.debug`: Set logging level.
- `--axon.port`: Desired port number.
- `--axon.external_port`: External port to advertise (for port forwarding on cloud providers like Vast.ai).

### Running on Cloud Providers with Port Forwarding (Vast.ai, etc.)

When running on cloud instances with port forwarding (like Vast.ai), you need to configure both internal and external ports:

1. **Choose your ports:**
   - Internal port: The port your miner listens on inside the instance (e.g., `14001`)
   - External port: The port that external clients connect to (assigned by your provider)

2. **Configure port forwarding in your provider:**
   - In Vast.ai: Set up port forwarding from external port (e.g., `23456`) to internal port `14001`

3. **Run the miner with both ports:**
   ```bash
   pm2 start neurons/miners/miner.py \
   --interpreter /usr/bin/python3 \
   --name desearch_miner \
   -- \
   --wallet.name <your-wallet-name> \
   --wallet.hotkey <your-hotkey> \
   --netuid 22 \
   --subtensor.network finney \
   --axon.port 14001 \
   --axon.external_port 23456
   ```

4. **Verify connectivity:**
   - External clients will connect to your external IP + external port
   - Bittensor network will advertise your external port for validator connections
   - Your miner listens internally on the internal port

**Example for Vast.ai:**
- Instance internal port: `14001`
- Vast.ai assigns external port: `34567`
- Command:
  ```bash
  pm2 start neurons/miners/miner.py --interpreter /usr/bin/python3 --name miner_1 -- --wallet.name miner --wallet.hotkey default --subtensor.network finney --netuid 22 --axon.port 14001 --axon.external_port 34567
  ```

- `--miner.name`: Path for miner data (miner.root / (wallet_cold - wallet_hot) / miner.name).
- `--miner.mock_dataset`: Set to True to use a mock dataset.
- `--miner.blocks_per_epoch`: Number of blocks until setting weights on chain.
- `--miner.openai_summary_model`: OpenAI model used for summarizing content. Default gpt-3.5-turbo-0125
- `--miner.openai_query_model`: OpenAI model used for generating queries. Default gpt-3.5-turbo-0125
- `--miner.openai_fix_query_model`: "OpenAI model used for fixing queries. Default gpt-4-1106-preview


## Conclusion
Following these steps, your desearch miner should be operational. Regularly monitor your processes and logs for any issues. For additional information or assistance, consult the official documentation or community resources.