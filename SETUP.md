# How to Push This to GitHub (Step by Step)

## 1. Create the Repository on GitHub

1. Go to https://github.com/new
2. Repository name: `tobeez-route-planner`
3. Description: `Internal last-mile route planning tool for Tobeez — FMCG distributor, Casablanca`
4. Set to **Public** (required for free GitHub Pages)
5. **Do NOT** check "Add README" (you already have one)
6. Click **Create repository**

---

## 2. Push from Your Computer

Open a terminal in the folder where you saved these files, then run:

```bash
git init
git add .
git commit -m "Initial commit — Tobeez Route Planner v1"
git branch -M main
git remote add origin https://github.com/YOURUSERNAME/tobeez-route-planner.git
git push -u origin main
```

> Replace `YOURUSERNAME` with your actual GitHub username.

---

## 3. Enable GitHub Pages (Live Demo)

1. Go to your repo on GitHub
2. Click **Settings** → **Pages** (left sidebar)
3. Under **Source**, select: `Deploy from a branch`
4. Branch: `main` / folder: `/ (root)`
5. Click **Save**

After ~60 seconds, your live demo will be at:
```
https://YOURUSERNAME.github.io/tobeez-route-planner
```

---

## 4. Update the README Links

In `README.md`, replace all instances of `yourusername` with your real GitHub username:

```bash
# On Mac/Linux
sed -i 's/yourusername/YOURUSERNAME/g' README.md

# Then commit
git add README.md
git commit -m "Update README with real links"
git push
```

---

## 5. Add to Your Profile (Optional but recommended)

If you have a GitHub profile README (`github.com/YOURUSERNAME/YOURUSERNAME`), add this to it:

```markdown
### 🚚 [Tobeez Route Planner](https://github.com/YOURUSERNAME/tobeez-route-planner)
Last-mile logistics tool I built as Product & Ops Lead at Tobeez.
Offline-capable, zero dependencies — custom XLSX parser, canvas map, NN + 2-opt optimizer.
[Live Demo →](https://YOURUSERNAME.github.io/tobeez-route-planner)
```

---

## 6. Future Updates

Whenever you improve the tool, just run:

```bash
cp /path/to/tobeez_planner.html ./tobeez_planner.html
git add tobeez_planner.html
git commit -m "Route planner: [describe what changed]"
git push
```

GitHub Pages will update the live demo automatically within ~30 seconds.

---

## Repo Structure

```
tobeez-route-planner/
├── tobeez_planner.html    ← The entire app (single file, ~90KB)
├── index.html             ← GitHub Pages redirect
├── README.md              ← Project writeup for your portfolio
├── .gitignore
├── SETUP.md               ← This file
└── screenshots/           ← Add screenshots here (drag into README later)
```
