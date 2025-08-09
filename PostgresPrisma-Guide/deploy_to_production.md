# 🚀 Deploy to Production Guide

## **C — Deploy to Production**

### 1️⃣ Initial Deployment (First-Time Setup on Production VPS)

#### A. Prepare Server
1. **SSH into VPS**
   ```bash
   ssh user@your_vps_ip
   ```
2. **Install dependencies**
   - Node.js & npm
   - PostgreSQL client
   - Any other required packages

3. **Clone your repository**
   ```bash
   git clone https://github.com/yourusername/yourrepo.git
   cd yourrepo
   ```

4. **Install npm packages**
   ```bash
   npm install
   ```

#### B. Database & Prisma Setup
1. **Set up `.env`** with your production database URL.
2. **Run Prisma migration** (creates schema & migration history):
   ```bash
   npx prisma migrate deploy
   ```
   > **Note:** Use `migrate deploy` in production — it applies already-created migrations without creating new ones.
3. **Generate Prisma client**:
   ```bash
   npx prisma generate
   ```

#### C. Start Application
Example using **PM2**:
```bash
pm2 start dist/index.js --name myapp
pm2 save
```

---

### 2️⃣ Nth-Time Deployment (Updates)

If you already have migrations from development:

1. **Push code to GitHub** from your local machine.
2. **On VPS**:
   ```bash
   ssh user@your_vps_ip
   cd yourrepo
   git pull origin main
   npm install
   ```
3. **Run migrations** (apply new ones only):
   ```bash
   npx prisma migrate deploy
   ```
4. **Generate updated Prisma client**:
   ```bash
   npx prisma generate
   ```
5. **Restart server**:
   ```bash
   pm2 restart myapp
   ```

---
✅ **Summary:**
- **First-Time:** Setup server → configure DB → deploy migrations → start app.
- **Nth-Time:** Pull updates → run migrations → restart app.
