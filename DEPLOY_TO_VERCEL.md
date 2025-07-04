# Deploy to Vercel - Step by Step

Since you need to authenticate with Vercel, please follow these steps:

## Option 1: Using Vercel CLI (Recommended)

1. **Login to Vercel:**
   ```bash
   vercel login
   ```
   This will open your browser to authenticate.

2. **Deploy the app:**
   ```bash
   cd kelsity-web-deploy
   vercel --prod
   ```

3. **Follow the prompts:**
   - Set up and deploy? **Y**
   - Which scope? **Select your account**
   - Link to existing project? **N** (unless you have one)
   - Project name? **kelsity** (or keep default)
   - Directory? **./** (current directory)
   - Override settings? **N**

## Option 2: Using Vercel Dashboard (Alternative)

1. Go to https://vercel.com
2. Click "Add New" → "Project"
3. Import from Git or upload the `kelsity-web-deploy` folder
4. Make sure to include the `vercel.json` file
5. Deploy

## Option 3: Using GitHub Integration

1. Push the web build to a GitHub repository
2. Connect Vercel to your GitHub
3. Import the repository
4. Vercel will auto-detect the `vercel.json` and deploy

## After Deployment

1. **Update your domain (if using custom domain):**
   - In Vercel dashboard → Settings → Domains
   - Add `kelsity.com` and `www.kelsity.com`

2. **Verify the fix:**
   - Visit: `https://your-app.vercel.app/verify-email?token=test`
   - Should show your Flutter app (not 404)

## Quick Deploy Script

Create this script for future deployments:

```bash
#!/bin/bash
# deploy.sh
echo "Building Flutter web..."
cd ..
flutter build web

echo "Copying to deployment folder..."
cp -r build/web/* kelsity-web-deploy/

echo "Deploying to Vercel..."
cd kelsity-web-deploy
vercel --prod

echo "Deployment complete!"
```

The `vercel.json` file is already in place and will handle all routing correctly!