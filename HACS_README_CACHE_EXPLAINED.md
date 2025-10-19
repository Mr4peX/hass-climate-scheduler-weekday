# HACS README Cache - Why Changes Don't Appear Immediately

## 🔍 The Problem

You updated the README.md on GitHub, but when you check HACS in Home Assistant, the **old README content still appears**. The new sections with "📚 Examples & Guides" and "📦 What Gets Installed via HACS" are missing.

## ⏰ The Answer: HACS Cache Timing

According to official HACS documentation, **custom repositories** (like yours) have specific update intervals:

### Update Schedule for Custom Repositories

| Trigger | Timing | Action |
|---------|--------|--------|
| **Startup** | When HA restarts | Fetches repo data via GitHub API |
| **Every 48 hours** | Automatic | Refreshes custom repository data via GitHub API |
| **Manual: "Update information"** | On demand | Immediately fetches latest data via GitHub API |
| **Manual: Browse repository** | When opened | Fetches latest data for that specific repo |

**Source:** https://hacs.xyz/docs/faq/data_sources/

## 📖 Why Your README Isn't Updated Yet

Your repository is installed as a **custom repository** in HACS. This means:

1. **HACS caches the README** from GitHub
2. **Cache refreshes every 48 hours** (not instantly)
3. **Last update was less than 48 hours ago** → showing old README
4. **GitHub shows correct README** (because no cache)

## ✅ How to Force README Update in HACS

### Method 1: Manual Update (FASTEST - 5 seconds)

1. Open **Home Assistant** → **HACS**
2. Find **"Climate Scheduler Weekday"** in your integrations
3. Click the **3-dot menu** (⋮) on the repository
4. Click **"Update information"** (or "Redownload")
5. Wait 5-10 seconds
6. Click on the repository again
7. **README should now show the new content!**

### Method 2: Browse Repository (FAST - 10 seconds)

1. Open **HACS**
2. Click on **"Climate Scheduler Weekday"**
3. This action triggers GitHub API fetch
4. README updates automatically
5. Refresh the page if needed

### Method 3: Restart Home Assistant (SLOW - 1-2 minutes)

1. Go to **Settings** → **System** → **Restart**
2. Wait for restart
3. Open HACS
4. README will be updated with latest content

### Method 4: Wait 48 Hours (AUTOMATIC)

- Do nothing
- HACS will automatically refresh after 48 hours
- Not recommended if you want to see changes now

## 🎯 Expected Result After Update

Once you force the update, your HACS README should show:

```
📚 Examples & Guides

💡 Important: HACS installs only the integration code (clean and minimal). 
Examples and guides are available on GitHub for you to browse, read, and 
download as needed. This keeps your Home Assistant installation clean while 
providing comprehensive documentation when you need it.

New to Climate Scheduler? Start here! → Browse Examples on GitHub

We've created comprehensive guides and real-world examples to help you get started:

📊 CSV Template - Edit schedules in Excel/LibreOffice (much easier than YAML!)
🏠 Complete System - Production config with 252 schedule entries (4 rooms, 8 thermostats)
🔄 Hybrid External Sensors - Prevent 2-hour timeout issue with keep-alive automations
📅 Three-Profile System - Weekly/Away/Holiday modes with automatic switching
📖 Step-by-Step Deployment - Beginner to advanced guides with testing procedures

→ Browse Examples on GitHub | → Quick Start Guide | → Download CSV Template

📦 What Gets Installed via HACS

HACS installs only the core integration to keep your system clean:

✅ custom_components/climate_scheduler/ (3 Python files, ~50KB)
❌ Examples folder is not installed (browse on GitHub instead)
❌ Documentation is not installed (read online)

All examples and guides remain on GitHub for easy access without cluttering 
your Home Assistant installation!
```

## 📝 Technical Details

### How HACS Fetches Repository Information

**For Default Repositories:**
- Uses HACS Data service (data-v2.hacs.xyz)
- Updates every 6 hours
- Faster cache refresh

**For Custom Repositories (YOUR CASE):**
- Uses GitHub REST API directly
- Updates every 48 hours
- Manual refresh available via "Update information"
- Cache is instance-specific (your HACS installation only)

### Why This Design?

1. **Reduces GitHub API rate limits** - GitHub limits API calls (5,000/hour for authenticated)
2. **Improves performance** - No need to fetch data constantly
3. **Saves bandwidth** - Home Assistant doesn't constantly ping GitHub
4. **Balances freshness vs efficiency** - 48 hours is reasonable for stable repos

## 🚀 Verification Steps

After forcing the update:

1. ✅ Open HACS → Climate Scheduler Weekday
2. ✅ Scroll down to README section
3. ✅ Look for "📚 Examples & Guides" heading
4. ✅ Look for "📦 What Gets Installed via HACS" section
5. ✅ Verify prominent notice box is visible
6. ✅ Check links point to GitHub examples folder

## 🌟 Key Takeaway

**HACS README cache is NORMAL behavior, not a bug!**

- GitHub shows instant updates ✅
- HACS shows cached version (48h refresh) ⏰
- Force refresh via "Update information" 🔄
- Your users will see updates within 48h or after manual refresh 👥

## 📚 Official HACS Documentation References

- **Data Sources:** https://hacs.xyz/docs/faq/data_sources/
- **Custom Repositories:** https://hacs.xyz/docs/faq/custom_repositories/
- **Troubleshooting:** https://hacs.xyz/docs/use/troubleshooting/logs/

## ✨ Current Status

- ✅ GitHub README: **Updated with new sections**
- ⏳ HACS README: **Will update within 48h or after manual refresh**
- ✅ Installation: **HACS installs only custom_components/**
- ✅ Examples: **Accessible on GitHub only**
- ✅ Version: **1.0.0 with Git tag**

---

**Summary:** Your README update is working perfectly! HACS just needs to refresh its cache. Use "Update information" in HACS to see changes immediately, or wait up to 48 hours for automatic refresh. This is standard HACS behavior for all custom repositories.
