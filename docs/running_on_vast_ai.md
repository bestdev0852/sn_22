# Running Bittensor Miner on Vast.ai

This guide explains how to run a Bittensor Subnet 22 (Desearch) miner on Vast.ai GPU instances with proper port configuration.

## Vast.ai Setup

### 1. Launch Instance
- Select a GPU instance (RTX 3060 or better recommended)
- Choose Ubuntu 20.04+ as the OS
- Ensure at least 8GB RAM, 4 CPU cores, 50GB SSD

### 2. Configure Ports
Vast.ai provides 2 ports by default. You need to set up port forwarding:

1. **In Vast.ai interface:**
   - Go to your instance details
   - Find the "Port Forwarding" or "Network" section
   - Note your assigned external ports (e.g., `23456` and `34567`)

2. **Choose internal ports:**
   - Miner port: `14001` (or any unused port)
   - Map one external port to this internal port

### 3. Connect to Instance
```bash
# SSH into your Vast.ai instance
ssh -p <external_ssh_port> root@<instance_ip>
```

### 4. Install Dependencies
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python 3.10+
sudo apt install python3.10 python3.10-venv python3-pip -y

# Install PM2
sudo apt install npm -y
sudo npm install pm2 -g
pm2 update

# Install Redis (optional, for caching)
sudo apt install redis-server -y
sudo systemctl start redis-server
```

### 5. Setup Miner
```bash
# Clone repository
git clone https://github.com/Desearch-ai/subnet-22.git
cd subnet-22

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
pip install -e .
```

### 6. Configure Environment Variables
```bash
# Create .env file
cat > .env << EOF
OPENAI_API_KEY=your_openai_key
TWITTER_BEARER_TOKEN=your_twitter_token
WANDB_API_KEY=your_wandb_key
APIFY_API_KEY=your_apify_key
SERPAPI_API_KEY=your_serpapi_key
EOF

# Load environment variables
export $(cat .env | xargs)
```

### 7. Create/Register Wallet
```bash
# Install Bittensor CLI (if not already available)
pip install bittensor

# Create wallet (if not exists)
btcli new_coldkey --wallet.name miner
btcli new_hotkey --wallet.name miner --wallet.hotkey default

# Register to subnet 22
btcli register --wallet.name miner --wallet.hotkey default --netuid 22 --subtensor.network finney
```

### 8. Run Miner with Port Configuration

**Important:** Replace the ports with your actual Vast.ai assigned ports.

```bash
# Example: Internal port 14001, External port 23456
pm2 start neurons/miners/miner.py \
--interpreter python \
--name desearch_miner \
-- \
--wallet.name miner \
--wallet.hotkey default \
--netuid 22 \
--subtensor.network finney \
--axon.port 14001 \
--axon.external_port 23456
```

### 9. Verify Setup
```bash
# Check PM2 status
pm2 status

# Check logs
pm2 logs desearch_miner

# Test connectivity (from external machine)
curl http://<vast_ai_external_ip>:<external_port>/health
```

## Port Configuration Details

### Why Two Ports Matter
- **Internal Port** (`--axon.port`): The port your miner application listens on inside the Vast.ai container
- **External Port** (`--axon.external_port`): The port that external Bittensor nodes use to connect to your miner

### Vast.ai Port Forwarding
1. Vast.ai assigns you external ports (visible in instance details)
2. You configure which external port forwards to which internal port
3. Bittensor needs to know both:
   - Where to listen internally (internal port)
   - What external port to advertise to the network

### Example Configuration
```
Vast.ai Instance:
- External IP: 123.45.67.89
- Assigned ports: 23456, 34567

Your Setup:
- Internal port: 14001
- External port: 23456 (forwarded to 14001)

Command:
pm2 start neurons/miners/miner.py ... --axon.port 14001 --axon.external_port 23456

Result:
- Miner listens on 14001 inside container
- External clients connect to 123.45.67.89:23456
- Bittensor network knows to route traffic to port 23456
```

## Troubleshooting

### Port Already in Use
```bash
# Check what's using the port
sudo netstat -tulpn | grep :14001

# Kill process if needed
sudo kill -9 <PID>
```

### Connection Issues
```bash
# Check if port is open
sudo ufw status
sudo ufw allow 14001

# Test internal connectivity
curl http://localhost:14001/health
```

### Bittensor Registration Issues
```bash
# Check wallet registration
btcli overview --wallet.name miner --subtensor.network finney

# Check axon registration
btcli axons --netuid 22 --subtensor.network finney
```

### PM2 Issues
```bash
# Restart PM2
pm2 restart desearch_miner

# Check logs
pm2 logs desearch_miner --lines 100

# Delete and restart
pm2 delete desearch_miner
pm2 start neurons/miners/miner.py [your_args]
```

## Performance Optimization

### For Vast.ai RTX 3060
- The GPU isn't heavily used by miners (mostly API calls)
- Focus on CPU cores (16+ recommended) and network speed
- Monitor with timing logs (see timing_metrics_guide.md)

### Cost Optimization
- Use spot instances when available
- Set up auto-shutdown for inactive periods
- Monitor usage to avoid overage charges

## Monitoring

### Basic Monitoring
```bash
# PM2 monitoring
pm2 monit

# System resources
htop
nvidia-smi

# Network
iftop
```

### Advanced Monitoring
- Enable WandB logging with `WANDB_API_KEY`
- Check timing metrics in `/tmp/desearch_miner/timing_metrics.jsonl`
- Monitor Redis cache hit rates

## Security Notes

- Never expose SSH on default port 22
- Use strong passwords/keys
- Regularly update the instance
- Monitor for unauthorized access
- Don't store private keys in plain text

## Support

If you encounter issues:
1. Check the logs: `pm2 logs desearch_miner`
2. Verify port configuration in Vast.ai interface
3. Test connectivity from external machine
4. Check Bittensor network status: `btcli overview --netuid 22`