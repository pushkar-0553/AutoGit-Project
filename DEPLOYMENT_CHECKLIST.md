# ✅ Deployment Checklist

## 📋 Pre-Deployment

### **1. MongoDB Atlas**
- [ ] Create MongoDB Atlas account
- [ ] Create free M0 cluster
- [ ] Create database user (username + password)
- [ ] Whitelist all IPs (0.0.0.0/0)
- [ ] Copy connection string
- [ ] Replace `<password>` in connection string

### **2. GitHub OAuth**
- [ ] Go to GitHub Developer Settings
- [ ] Create OAuth App (if not exists)
- [ ] Note down Client ID
- [ ] Note down Client Secret
- [ ] Keep these for later

### **3. Prepare Code**
- [ ] All code committed locally
- [ ] `.env` files are in `.gitignore`
- [ ] `node_modules/` in `.gitignore`
- [ ] No sensitive data in code

---

## 🔧 Backend Deployment (Render)

### **Step 1: Push to GitHub**
```bash
cd backend
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/autogit-backend.git
git push -u origin main
```

- [ ] Backend code pushed to GitHub
- [ ] Repository is public or Render has access

### **Step 2: Deploy to Render**
- [ ] Sign up at render.com
- [ ] Click "New +" → "Web Service"
- [ ] Connect GitHub repository
- [ ] Select `autogit-backend` repo
- [ ] Configure:
  - Name: `autogit-backend`
  - Region: Oregon
  - Branch: `main`
  - Build Command: `npm install`
  - Start Command: `npm start`
  - Plan: Free

### **Step 3: Add Environment Variables**
In Render dashboard → Environment:

- [ ] `MONGODB_URI` = Your MongoDB connection string
- [ ] `GITHUB_CLIENT_ID` = Your GitHub OAuth Client ID
- [ ] `GITHUB_CLIENT_SECRET` = Your GitHub OAuth Client Secret
- [ ] `SESSION_SECRET` = Random string (generate one)
- [ ] `CLIENT_URL` = `https://your-app.vercel.app` (update after frontend deploy)
- [ ] `NODE_ENV` = `production`

### **Step 4: Deploy & Test**
- [ ] Click "Create Web Service"
- [ ] Wait for deployment (5-10 minutes)
- [ ] Note your backend URL: `https://autogit-backend.onrender.com`
- [ ] Test health check: Visit `https://your-backend.onrender.com/api/health`
- [ ] Should see: `{"status":"OK","message":"AutoGit API is running"}`

---

## 🎨 Frontend Deployment (Vercel)

### **Step 1: Update API Configuration**

Create `frontend/.env.production`:
```env
REACT_APP_API_URL=https://your-backend.onrender.com
```

- [ ] Created `.env.production` file
- [ ] Added correct backend URL

### **Step 2: Push to GitHub**
```bash
cd frontend
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/autogit-frontend.git
git push -u origin main
```

- [ ] Frontend code pushed to GitHub
- [ ] Repository is public or Vercel has access

### **Step 3: Deploy to Vercel**
- [ ] Sign up at vercel.com
- [ ] Click "Add New..." → "Project"
- [ ] Import `autogit-frontend` repo
- [ ] Configure:
  - Framework: Create React App
  - Build Command: `npm run build`
  - Output Directory: `build`

### **Step 4: Add Environment Variables**
In Vercel dashboard → Settings → Environment Variables:

- [ ] `REACT_APP_API_URL` = Your Render backend URL

### **Step 5: Deploy & Test**
- [ ] Click "Deploy"
- [ ] Wait for deployment (2-5 minutes)
- [ ] Note your frontend URL: `https://autogit-frontend.vercel.app`
- [ ] Visit your frontend URL
- [ ] Should see login page

---

## 🔄 Final Configuration

### **Step 1: Update GitHub OAuth**
Go to GitHub Developer Settings → Your OAuth App:

- [ ] Homepage URL = `https://your-app.vercel.app`
- [ ] Authorization callback URL = `https://your-backend.onrender.com/api/auth/github/callback`
- [ ] Click "Update application"

### **Step 2: Update Backend Environment**
In Render dashboard → Environment:

- [ ] Update `CLIENT_URL` = Your actual Vercel URL
- [ ] Click "Save Changes"
- [ ] Wait for redeploy

### **Step 3: Update Frontend Environment**
In Vercel dashboard → Settings → Environment Variables:

- [ ] Verify `REACT_APP_API_URL` = Your actual Render URL
- [ ] Redeploy if needed

---

## ✅ Testing

### **Test Backend**
- [ ] Visit: `https://your-backend.onrender.com/api/health`
- [ ] Should return: `{"status":"OK",...}`

### **Test Frontend**
- [ ] Visit: `https://your-app.vercel.app`
- [ ] Should see login page
- [ ] No console errors

### **Test Authentication**
- [ ] Click "Login with GitHub"
- [ ] Should redirect to GitHub
- [ ] Authorize the app
- [ ] Should redirect back to your app
- [ ] Should see dashboard

### **Test Full Flow**
- [ ] Login successful
- [ ] Dashboard loads
- [ ] Can see repositories
- [ ] Go to Workspace
- [ ] Select repository
- [ ] Can create/edit files
- [ ] Can save to GitHub
- [ ] Changes appear in GitHub

---

## 🐛 Troubleshooting

### **Backend Issues**

**502 Bad Gateway:**
- Check Render logs
- Verify MongoDB connection string
- Check environment variables

**CORS Errors:**
- Verify `CLIENT_URL` in Render
- Check it matches your Vercel URL exactly
- Include `https://`

**OAuth Fails:**
- Check GitHub OAuth callback URL
- Must match Render URL exactly
- Check Client ID and Secret in Render

### **Frontend Issues**

**API Calls Fail:**
- Check `REACT_APP_API_URL` in Vercel
- Verify backend URL is correct
- Check backend is running

**Blank Page:**
- Check browser console for errors
- Verify build succeeded in Vercel
- Check `vercel.json` exists

**Routes Don't Work:**
- Ensure `vercel.json` is configured
- Check routing configuration

### **Database Issues**

**Connection Timeout:**
- Check IP whitelist in MongoDB Atlas
- Should have 0.0.0.0/0
- Check connection string format

**Authentication Failed:**
- Verify username/password
- Check connection string
- Ensure user has read/write permissions

---

## 📝 Important URLs

Save these for reference:

```
Frontend URL: https://_____________________.vercel.app
Backend URL:  https://_____________________.onrender.com
MongoDB:      https://cloud.mongodb.com
GitHub OAuth: https://github.com/settings/developers
```

---

## 🎊 Success Criteria

Your deployment is successful when:

- ✅ Backend health check responds
- ✅ Frontend loads without errors
- ✅ Login with GitHub works
- ✅ Dashboard shows repositories
- ✅ Can navigate to Workspace
- ✅ Can select repository
- ✅ Can create/edit files
- ✅ Changes save to GitHub
- ✅ No console errors
- ✅ All features work

---

## 💡 Tips

1. **First Deploy:** Backend takes 5-10 min, Frontend takes 2-5 min
2. **Free Tier:** Backend sleeps after 15 min of inactivity
3. **Wake Up:** First request after sleep takes 30-60 seconds
4. **Logs:** Check Render logs for backend errors
5. **Redeploy:** Push to GitHub to trigger redeploy
6. **Environment:** Changes to env vars require redeploy
7. **Custom Domain:** Can add later in Vercel settings
8. **HTTPS:** Automatic with both Render and Vercel

---

## 🚀 You're Ready!

Follow this checklist step by step, and your AutoGit app will be live!

**Good luck!** 🎉
