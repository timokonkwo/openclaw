# Deploying OpenClaw "Mance" to AWS Free Tier

This guide outlines how to deploy your OpenClaw agent on an AWS EC2 instance using the Free Tier (t2.micro or t3.micro).

## Prerequisites
- AWS Account.
- SSH Key Pair for EC2.
- Basic familiarity with terminal commands.

## Step 1: Launch EC2 Instance
1. **Log in to AWS Console** and navigate to **EC2**.
2. Click **Launch Instance**.
3. **Name:** `OpenClaw-Mance`.
4. **OS Image:** Ubuntu Server 24.04 LTS (Free tier eligible).
5. **Instance Type:** `t3.micro` (or `t2.micro` if t3 is unavailable in your region).
6. **Key Pair:** Create new or select existing (save the `.pem` file!).
7. **Network Settings:**
   - Allow SSH traffic from "My IP" (for security) or "Anywhere".
   - Allow HTTP/HTTPS traffic (optional, if using webhooks).
8. **Storage:** Set to 20-30GB gp3 (Free tier allows up to 30GB).
9. **Launch Instance**.

## Step 2: Connect to Instance
Open your terminal and SSH into the instance:
```bash
chmod 400 your-key.pem
ssh -i "your-key.pem" ubuntu@<public-ip-address>
```

## Step 3: Install Dependencies (Docker Method)
The easiest way to run OpenClaw is via Docker.

1. **Update System:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **Install Docker:**
   ```bash
   sudo apt install -y docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker newuser # Optional: run docker without sudo
   # You may need to logout and login again for group changes to take effect
   ```

## Step 4: Deploy OpenClaw

### Option A: Clone & Build
1. **Clone your repo:**
   ```bash
   git clone https://github.com/your-repo/openclaw.git
   cd openclaw
   ```
2. **Configure Environment:**
   Create a `.env` file with your credentials:
   ```bash
   nano .env
   ```
   *Paste your keys (Telegram Token, Anthropic Key, etc.) here.*
3. **Build & Run with Docker Compose:**
   ```bash
   docker compose up -d --build
   ```

### Option B: Manual Node.js Setup (Conserves Resources)
If Docker is too heavy for the micro instance:
1. **Install Node.js (v22):**
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
   sudo apt install -y nodejs
   ```
2. **Install pnpm:**
   ```bash
   npm install -g pnpm
   ```
3. **Install Dependencies:**
   ```bash
   pnpm install
   ```
4. **Start OpenClaw:**
   Use PM2 to keep it running in the background.
   ```bash
   npm install -g pm2
   pm2 start "pnpm start" --name openclaw
   pm2 save
   pm2 startup
   ```

## Step 5: Post-Deployment
- **Verify:** Check logs with radio silence? No! Check logs!
  - Docker: `docker logs -f openclaw`
  - PM2: `pm2 logs openclaw`
- **Security:** Ensure your `.env` file is secure. DO NOT commit it to public repos.

## Step 6: Ongoing Management
- **Updates:** `git pull` then rebuild/restart.
- **Monitoring:** Check CPU/RAM usage to ensure you stay within Free Tier limits.
