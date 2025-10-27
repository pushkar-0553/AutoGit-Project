# 🚀 Quick Deploy Guide

## ⚡ 30-Minute Deployment

Follow these steps to deploy AutoGit in ~30 minutes.

---

## 1️⃣ MongoDB Atlas (5 minutes)

```
1. Go to: https://www.mongodb.com/cloud/atlas/register
2. Sign up (free)
3. Create cluster (M0 Free)
4. Create user: autogit-user
5. Whitelist IP: 0.0.0.0/0
6. Get connection string
7. Save it!
```

**Connection String Format:**
```
mongodb+srv://autogit-user:PASSWORD@cluster.xxxxx.mongodb.net/autogit
```

---

## 2️⃣ Backend to Render (10 minutes)

### **Push to GitHub:**
```bash
cd backend
git init
git add .
git commit -m "Deploy backend"
git branch -M main
git remote add origin https://github.com/USERNAME/autogit-backend.git
git push -u origin main
```

### **Deploy:**
```
1. Go to: https://render.com
2. Sign up with GitHub
3. New + → Web Service
4. Connect autogit-backend repo
5. Settings:
   - Build: npm install
   - Start: npm start
   - Plan: Free
6. Add Environment Variables:
   - MONGODB_URI
   - GITHUB_CLIENT_ID
   - GITHUB_CLIENT_SECRET
   - SESSION_SECRET (random string)
   - CLIENT_URL (add after frontend)
   - NODE_ENV=production
7. Create Web Service
8. Wait 5-10 minutes
9. Save URL: https://YOUR-APP.onrender.com
```

---

## 3️⃣ Frontend to Vercel (10 minutes)

### **Create .env.production:**
```bash
cd frontend
echo "REACT_APP_API_URL=https://YOUR-BACKEND.onrender.com" > .env.production
```

### **Push to GitHub:**
```bash
git init
git add .
git commit -m "Deploy frontend"
git branch -M main
git remote add origin https://github.com/USERNAME/autogit-frontend.git
git push -u origin main
```

### **Deploy:**
```
1. Go to: https://vercel.com
2. Sign up with GitHub
3. New Project
4. Import autogit-frontend repo
5. Settings:
   - Framework: Create React App
   - Build: npm run build
   - Output: build
6. Add Environment Variable:
   - REACT_APP_API_URL=https://YOUR-BACKEND.onrender.com
7. Deploy
8. Wait 2-5 minutes
9. Save URL: https://YOUR-APP.vercel.app
```

---

## 4️⃣ Update GitHub OAuth (5 minutes)

```
1. Go to: https://github.com/settings/developers
2. Select your OAuth App
3. Update:
   - Homepage: https://YOUR-APP.vercel.app
   - Callback: https://YOUR-BACKEND.onrender.com/api/auth/github/callback
4. Save
```

---

## 5️⃣ Update Backend URL (2 minutes)

```
1. Go to Render dashboard
2. Select autogit-backend
3. Environment tab
4. Update CLIENT_URL:
   - CLIENT_URL=https://YOUR-APP.vercel.app
5. Save (auto-redeploys)
```

---

## ✅ Test (3 minutes)

### **Backend:**
Visit: `https://YOUR-BACKEND.onrender.com/api/health`

Should see:
```json
{"status":"OK","message":"AutoGit API is running"}
```

### **Frontend:**
1. Visit: `https://YOUR-APP.vercel.app`
2. Click "Login with GitHub"
3. Authorize
4. Should see dashboard

### **Full Test:**
1. Go to Workspace
2. Select repository
3. Create file
4. Save
5. Check GitHub - file should be there!

---

## 🎊 Done!

Your AutoGit is now live at:
- **Frontend:** `https://YOUR-APP.vercel.app`
- **Backend:** `https://YOUR-BACKEND.onrender.com`

---

## 📝 Save These

```
MongoDB: _________________________________
Backend:  _________________________________
Frontend: _________________________________
```

---

## 🐛 Issues?

**Backend 502 Error:**
- Check Render logs
- Verify MongoDB connection string

**CORS Error:**
- Check CLIENT_URL in Render
- Must match Vercel URL exactly

**OAuth Fails:**
- Check GitHub callback URL
- Must match Render URL exactly

**Frontend Blank:**
- Check browser console
- Verify REACT_APP_API_URL

---

## 💡 Tips

- Backend sleeps after 15 min (free tier)
- First request after sleep: 30-60 sec
- Check logs in Render/Vercel dashboards
- Push to GitHub to redeploy
- Environment changes need redeploy

---

**Happy deploying!** 🚀
