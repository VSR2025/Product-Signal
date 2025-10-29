# GitHub Setup Instructions

## Quick Start (5 minutes)

### 1. Create the Repo on GitHub
1. Go to github.com and click "New Repository"
2. Name: `llm-trace-review-tool` (or whatever you prefer)
3. Description: "Design specification for an LLM trace review interface based on HCI principles"
4. Make it **Public** (so you can share the link easily)
5. **Don't** initialize with README (we have one)
6. Click "Create repository"

### 2. Push Your Files

Open terminal and navigate to where you downloaded the `github-repo` folder, then:

```bash
cd github-repo

# Initialize git
git init

# Add all files
git add .

# First commit
git commit -m "Initial commit: LLM trace review tool specification"

# Add your GitHub repo as remote (replace YOUR_USERNAME)
git remote add origin https://github.com/YOUR_USERNAME/llm-trace-review-tool.git

# Push to GitHub
git branch -M main
git push -u origin main
```

### 3. Customize the README

Before pushing, update `README.md`:
- Line 96: Add your name
- Line 98: Add your contact info (Discord, Twitter, LinkedIn, etc.)

### 4. Share the Link!

Your repo will be at:
```
https://github.com/YOUR_USERNAME/llm-trace-review-tool
```

Drop that in Discord and people can browse everything!

---

## Repo Structure (What You're Uploading)

```
llm-trace-review-tool/
├── README.md                          # Main overview (what people see first)
├── .gitignore                         # Keeps actual trace data private
├── docs/
│   └── full_specification.md          # Complete design doc (14 features)
├── lovable/
│   ├── prompt.md                      # Natural language for Lovable
│   └── spec.json                      # Structured architecture
└── data/
    └── sample_traces.csv              # Sample data format (3 examples)
```

---

## Optional: Add Your Actual Traces (Careful!)

If your trace data isn't sensitive:
1. Copy `traces.csv` to `data/` folder
2. Remove `data/traces.csv` from `.gitignore`
3. Commit and push

If it IS sensitive:
- Keep using `sample_traces.csv` only
- `.gitignore` will prevent accidental upload of real data

---

## Tips

**Good commit messages:**
- "Initial commit: Complete specification"
- "Update README with contact info"
- "Add Phase 2 features to spec"

**Keeping it updated:**
```bash
# After making changes
git add .
git commit -m "Your message here"
git push
```

**If you want a cleaner URL:**
- Enable GitHub Pages in repo settings
- Your spec will be at: `username.github.io/llm-trace-review-tool`

---

## Troubleshooting

**"fatal: remote origin already exists"**
```bash
git remote remove origin
git remote add origin https://github.com/YOUR_USERNAME/llm-trace-review-tool.git
```

**Need to update remote URL:**
```bash
git remote set-url origin https://github.com/YOUR_USERNAME/llm-trace-review-tool.git
```

**Want to start over:**
```bash
rm -rf .git
git init
# Then follow Step 2 again
```
