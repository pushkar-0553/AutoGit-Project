# 🚀 AutoGit Deployment Guide

## 📋 Deployment Stack

- **Database:** MongoDB Atlas (Free Tier)
- **Backend:** Render (Free Tier)
- **Frontend:** Vercel (Free Tier)

---

## 🗂️ Part 1: MongoDB Atlas Setup

### **Step 1: Create MongoDB Atlas Account**

1. Go to [https://www.mongodb.com/cloud/atlas/register](https://www.mongodb.com/cloud/atlas/register)
2. Sign up for free account
3. Choose **FREE** tier (M0 Sandbox)

### **Step 2: Create Cluster**

1. Click **"Build a Database"**
2. Choose **FREE** tier (M0)
3. Select **Cloud Provider:** AWS
4. Select **Region:** Closest to you
5. Cluster Name: `autogit-cluster`
6. Click **"Create"**

### **Step 3: Create Database User**

1. Go to **Database Access** (left sidebar)
2. Click **"Add New Database User"**
3. Authentication Method: **Password**
4. Username: `autogit-user`
5. Password: Generate secure password (save it!)
6. Database User Privileges: **Read and write to any database**
7. Click **"Add User"**

### **Step 4: Whitelist IP Addresses**

1. Go to **Network Access** (left sidebar)
2. Click **"Add IP Address"**
3. Click **"Allow Access from Anywhere"** (0.0.0.0/0)
4. Click **"Confirm"**

⚠️ **Note:** For production, restrict to specific IPs

### **Step 5: Get Connection String**

1. Go to **Database** → **Connect**
2. Choose **"Connect your application"**
3. Driver: **Node.js**
4. Copy the connection string:
   ```
   mongodb+srv://autogit-user:<password>@autogit-cluster.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
5. Replace `<password>` with your actual password
6. **Save this connection string!**

---

## 🔧 Part 2: Prepare Backend for Deployment

### **Step 1: Update Environment Variables**

Create `backend/.env.production`:

```env
# MongoDB Atlas
MONGODB_URI=mongodb+srv://autogit-user:YOUR_PASSWORD@autogit-cluster.xxxxx.mongodb.net/autogit?retryWrites=true&w=majority

# GitHub OAuth (from GitHub Developer Settings)
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret

# Session Secret (generate random string)
SESSION_SECRET=your_super_secret_random_string_here

# URLs
CLIENT_URL=https://your-app.vercel.app
BACKEND_URL=https://your-app.onrender.com

# Port
PORT=10000
```

### **Step 2: Update GitHub OAuth Callback**

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Select your OAuth App
3. Update **Authorization callback URL:**
   ```
   https://your-app.onrender.com/api/auth/github/callback
   ```
4. Update **Homepage URL:**
   ```
   https://your-app.vercel.app
   ```

### **Step 3: Create `render.yaml`**

Create `backend/render.yaml`:

```yaml
services:
  - type: web
    name: autogit-backend
    env: node
    region: oregon
    plan: free
    buildCommand: npm install
    startCommand: npm start
    envVars:
      - key: NODE_ENV
        value: production
      - key: MONGODB_URI
        sync: false
      - key: GITHUB_CLIENT_ID
        sync: false
      - key: GITHUB_CLIENT_SECRET
        sync: false
      - key: SESSION_SECRET
        sync: false
      - key: CLIENT_URL
        sync: false
```

### **Step 4: Update `backend/package.json`**

Add/update scripts:

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}
```

### **Step 5: Update CORS Configuration**

Update `backend/server.js`:

```javascript
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

## 🌐 Part 3: Deploy Backend to Render

### **Step 1: Push to GitHub**

```bash
cd backend
git init
git add .
git commit -m "Initial backend commit"
git branch -M main
git remote add origin https://github.com/yourusername/autogit-backend.git
git push -u origin main
```

### **Step 2: Create Render Account**

1. Go to [https://render.com](https://render.com)
2. Sign up with GitHub
3. Authorize Render

### **Step 3: Create New Web Service**

1. Click **"New +"** → **"Web Service"**
2. Connect your GitHub repository
3. Select `autogit-backend` repository
4. Configure:
   - **Name:** `autogit-backend`
   - **Region:** Oregon (US West)
   - **Branch:** `main`
   - **Root Directory:** Leave empty (or `backend` if monorepo)
   - **Runtime:** Node
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Plan:** Free

### **Step 4: Add Environment Variables**

In Render dashboard, go to **Environment** tab and add:

```
MONGODB_URI=mongodb+srv://autogit-user:PASSWORD@...
GITHUB_CLIENT_ID=your_client_id
GITHUB_CLIENT_SECRET=your_client_secret
SESSION_SECRET=your_random_secret
CLIENT_URL=https://your-app.vercel.app
NODE_ENV=production
```

### **Step 5: Deploy**

1. Click **"Create Web Service"**
2. Wait for deployment (5-10 minutes)
3. Your backend URL: `https://autogit-backend.onrender.com`

⚠️ **Note:** Free tier sleeps after 15 minutes of inactivity

---

## 🎨 Part 4: Prepare Frontend for Deployment

### **Step 1: Update API Base URL**

Create `frontend/.env.production`:

```env
REACT_APP_API_URL=https://autogit-backend.onrender.com
```

### **Step 2: Update Axios Configuration**

Create/update `frontend/src/config/api.js`:

```javascript
import axios from 'axios';

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000';

axios.defaults.baseURL = API_URL;
axios.defaults.withCredentials = true;

export default axios;
```

### **Step 3: Update All API Calls**

Replace all `axios.get('/api/...')` with:

```javascript
import axios from './config/api';

// Then use normally
axios.get('/api/tasks')
```

### **Step 4: Create `vercel.json`**

Create `frontend/vercel.json`:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "build"
      }
    }
  ],
  "routes": [
    {
      "src": "/static/(.*)",
      "dest": "/static/$1"
    },
    {
      "src": "/favicon.jpg",
      "dest": "/favicon.jpg"
    },
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

### **Step 5: Update `package.json`**

Add build script:

```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "vercel-build": "react-scripts build"
  }
}
```

---

## 🚀 Part 5: Deploy Frontend to Vercel

### **Step 1: Push to GitHub**

```bash
cd frontend
git init
git add .
git commit -m "Initial frontend commit"
git branch -M main
git remote add origin https://github.com/yourusername/autogit-frontend.git
git push -u origin main
```

### **Step 2: Create Vercel Account**

1. Go to [https://vercel.com](https://vercel.com)
2. Sign up with GitHub
3. Authorize Vercel

### **Step 3: Import Project**

1. Click **"Add New..."** → **"Project"**
2. Import `autogit-frontend` repository
3. Configure:
   - **Framework Preset:** Create React App
   - **Root Directory:** `./` (or `frontend` if monorepo)
   - **Build Command:** `npm run build`
   - **Output Directory:** `build`

### **Step 4: Add Environment Variables**

In Vercel dashboard, go to **Settings** → **Environment Variables**:

```
REACT_APP_API_URL=https://autogit-backend.onrender.com
```

### **Step 5: Deploy**

1. Click **"Deploy"**
2. Wait for deployment (2-5 minutes)
3. Your frontend URL: `https://autogit-frontend.vercel.app`

---

## 🔄 Part 6: Update OAuth Callbacks

### **Update GitHub OAuth App**

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Update your OAuth App:
   - **Homepage URL:** `https://autogit-frontend.vercel.app`
   - **Authorization callback URL:** `https://autogit-backend.onrender.com/api/auth/github/callback`

### **Update Backend Environment Variables**

In Render dashboard, update:
```
CLIENT_URL=https://autogit-frontend.vercel.app
```

### **Update Frontend Environment Variables**

In Vercel dashboard, update:
```
REACT_APP_API_URL=https://autogit-backend.onrender.com
```

---

## ✅ Part 7: Verification

### **Test Backend**

Visit: `https://autogit-backend.onrender.com/api/health`

Should return:
```json
{
  "status": "ok",
  "message": "Server is running"
}
```

### **Test Frontend**

1. Visit: `https://autogit-frontend.vercel.app`
2. Click **"Login with GitHub"**
3. Authorize app
4. Should redirect to dashboard

### **Test Full Flow**

1. Login
2. Go to Workspace
3. Select repository
4. Create/edit file
5. Save to GitHub

---

## 🐛 Troubleshooting

### **Backend Issues**

**Problem:** 502 Bad Gateway
- **Solution:** Check Render logs, ensure MongoDB connection string is correct

**Problem:** CORS errors
- **Solution:** Verify CLIENT_URL in Render environment variables

**Problem:** OAuth fails
- **Solution:** Check GitHub OAuth callback URL matches Render URL

### **Frontend Issues**

**Problem:** API calls fail
- **Solution:** Check REACT_APP_API_URL in Vercel environment variables

**Problem:** Blank page
- **Solution:** Check browser console, verify build succeeded

**Problem:** Routes don't work
- **Solution:** Ensure `vercel.json` is configured correctly

### **Database Issues**

**Problem:** Connection timeout
- **Solution:** Check MongoDB Atlas IP whitelist (0.0.0.0/0)

**Problem:** Authentication failed
- **Solution:** Verify username/password in connection string

---

## 💰 Cost Breakdown

| Service | Plan | Cost | Limits |
|---------|------|------|--------|
| **MongoDB Atlas** | M0 Free | $0 | 512MB storage |
| **Render** | Free | $0 | Sleeps after 15min |
| **Vercel** | Hobby | $0 | 100GB bandwidth |
| **Total** | | **$0/month** | |

---

## 🚀 Quick Deployment Checklist

- [ ] MongoDB Atlas cluster created
- [ ] Database user created
- [ ] Connection string saved
- [ ] Backend pushed to GitHub
- [ ] Backend deployed to Render
- [ ] Environment variables added to Render
- [ ] Frontend pushed to GitHub
- [ ] Frontend deployed to Vercel
- [ ] Environment variables added to Vercel
- [ ] GitHub OAuth callbacks updated
- [ ] Backend health check passes
- [ ] Frontend loads correctly
- [ ] Login flow works
- [ ] File operations work

---

## 📝 Important Notes

### **Free Tier Limitations**

**Render:**
- Sleeps after 15 minutes of inactivity
- First request after sleep takes 30-60 seconds
- 750 hours/month free

**Vercel:**
- 100GB bandwidth/month
- Unlimited deployments
- Automatic HTTPS

**MongoDB Atlas:**
- 512MB storage
- Shared CPU
- No backups on free tier

### **Production Recommendations**

For production use, consider:
- Render Starter plan ($7/month) - No sleep
- MongoDB M10 plan ($10/month) - Dedicated cluster
- Vercel Pro ($20/month) - More bandwidth
- Custom domain names
- SSL certificates (free with Vercel)

---

## 🎊 You're Done!

Your AutoGit app is now live at:
- **Frontend:** `https://autogit-frontend.vercel.app`
- **Backend:** `https://autogit-backend.onrender.com`
- **Database:** MongoDB Atlas

**Share your app with the world!** 🌍
