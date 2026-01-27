# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this blog repository.

## Repository Overview

This is a personal technical blog built with Jekyll and the Chirpy theme (~v7.1), hosted on GitHub Pages.

**Website:** https://pengguanya.github.io

**Content Focus:**
- Data Science, Statistics, Biomedical Development, Clinical Trials
- Linux/Unix systems and DevOps practices
- AI/ML applications and research
- Quantitative research and probabilistic modeling

**Blog Pillars:**
The blog is organized around four content categories that represent core professional areas:
- **Quantitative Research**: Mathematics, statistics, probabilistic modeling (Beta distributions, Bayesian inference)
- **Drug Development**: Clinical trial protocols, safety reporting, regulatory landscapes (EU AI Act, post-market investigation)
- **AI & ML**: Artificial Intelligence, Reinforcement Learning, LLM integration
- **DevOps & Computing**: Linux/Unix systems, cloud infrastructure (Azure), automated development workflows

## Repository Structure

```
pengguanya.github.io/
├── _config.yml              # Main site configuration
├── Gemfile                  # Ruby dependencies
├── _posts/                  # Published blog posts (YYYY-MM-DD-title.md)
├── _drafts/                 # Draft posts (no date prefix needed)
├── _tabs/                   # Static pages (about, archives, categories, tags)
│   └── about.md            # About page with blog pillars description
├── _data/                   # Site data files
│   ├── authors.yml         # Author information
│   ├── contact.yml         # Social links
│   └── share.yml           # Sharing options
├── assets/                  # Static assets
│   ├── headers/            # Blog post header images (YYYY-MM-DD-title.webp)
│   └── img/                # Other images
├── .github/workflows/       # CI/CD configuration
│   └── pages-deploy.yml    # GitHub Actions deployment workflow
├── .jekyll-cache/          # Jekyll cache (gitignored)
├── _site/                  # Generated static site (gitignored)
└── tools/                  # Development scripts
    ├── run.sh              # Built-in development server script
    └── test.sh             # Build and test script
```

## Development Environment

### Prerequisites
- **Ruby**: 3.3+ (required by GitHub Actions)
- **Bundler**: Ruby package manager
- **Jekyll**: Static site generator (installed via Bundler)
- **Custom tool**: `localhost` command (external script at `~/.local/bin/localhost`)

### Initial Setup

```bash
# Install Ruby dependencies
bundle install

# Verify installation
bundle exec jekyll --version
```

## Local Development

### Using the `localhost` Command (Recommended)

The `localhost` command is a custom bash script that provides convenient server management:

**Features:**
- Auto-kills conflicting processes on the target port
- Supports draft mode
- Configurable host and port
- Works with both Jekyll and Rails projects

**Basic Usage:**
```bash
# Start Jekyll server on default 127.0.0.1:4000
localhost

# Include draft posts
localhost --draft

# Custom port with drafts
localhost --port 4001 --draft

# Accessible from network (useful for mobile testing)
localhost --host 0.0.0.0 --port 8080

# Show help
localhost --help
```

**What It Does:**
1. Checks if port is in use and kills conflicting processes
2. Starts Jekyll with specified options
3. Server accessible at http://127.0.0.1:4000/ (or custom host/port)

### Alternative Development Methods

**Manual Jekyll Command:**
```bash
bundle exec jekyll serve              # Standard server
bundle exec jekyll serve --draft      # Include drafts
bundle exec jekyll serve --host 0.0.0.0 --port 8080  # Custom host/port
```

**Using Built-in Development Script:**
```bash
./tools/run.sh                        # Start development server
```

**VS Code Integration:**
- Use "Run Jekyll Server" task (Ctrl+Shift+B or Cmd+Shift+B)
- Configured in `.vscode/tasks.json`

### Building the Site

```bash
# Build to _site/ directory
bundle exec jekyll build

# Build and test with html-proofer
./tools/test.sh
```

## Creating Content

### New Blog Post Workflow

1. **Create post file** in `_posts/` with date prefix:
   ```bash
   touch _posts/2026-01-27-my-new-post.md
   ```

2. **Add frontmatter** with required fields:
   ```yaml
   ---
   title: "Your Post Title"
   date: 2026-01-27
   author: peng
   categories: [Single Category Name]
   tags: [tag1, tag2, tag3]
   math: false
   image:
     path: assets/headers/2026-01-27-my-new-post.webp
     alt: "Image description for accessibility"
   ---
   ```

3. **Write content** in Markdown below frontmatter

4. **Add header image** (optional but recommended):
   - Place image in `assets/headers/`
   - Naming convention: `YYYY-MM-DD-title.webp`
   - Format: WebP preferred for performance

5. **Preview locally**:
   ```bash
   localhost --draft  # If working on draft
   # Visit http://127.0.0.1:4000/
   ```

### Frontmatter Field Reference

**Required Fields:**
- `title`: Post title (quoted string)
- `date`: Publication date (YYYY-MM-DD format)
- `author`: Author identifier (typically "peng", defined in `_data/authors.yml`)
- `categories`: **MUST be single category** (see Category System Rules below)

**Optional Fields:**
- `tags`: Array of tags for secondary classification
- `math`: Set to `true` to enable MathJax for mathematical equations
- `image`: Header image configuration
  - `path`: Relative path to image file
  - `alt`: Alternative text for accessibility
- `pin`: Set to `true` to pin post to top of home page
- `toc`: Override global TOC setting (default: true)

### Category System Rules

**CRITICAL: Single Category Only**

Each post **MUST** have exactly ONE category from the four blog pillars:
- `Quantitative Research`
- `Drug Development`
- `AI & ML`
- `DevOps & Computing`

**Correct Usage:**
```yaml
categories: [AI & ML]                 # ✅ Single category
categories: [DevOps & Computing]      # ✅ Single category
```

**Incorrect Usage:**
```yaml
categories: [AI & ML, DevOps & Computing]  # ❌ Multiple categories
categories: ["AI", "ML"]                    # ❌ Creates nested folders
```

**Why Single Category?**
- Multiple categories create hierarchical folder structure in Jekyll: `/categories/AI/ML/`
- This breaks the flat category page structure desired for the blog
- Results in broken category links and confusing navigation

**Secondary Classification:**
Use `tags` for secondary topics:
```yaml
categories: [AI & ML]
tags: [azure, openai, llm, api, tutorial]  # ✅ Tags for secondary topics
```

**Jekyll Auto-Discovery:**
- Categories are auto-discovered from post frontmatter
- No need to register categories in `_config.yml`
- Category pages are auto-generated by `jekyll-archives` plugin

### Working with Drafts

**Creating Drafts:**
```bash
# Create draft (no date prefix needed)
touch _drafts/my-draft-post.md
```

**Draft Frontmatter:**
```yaml
---
title: "Draft Post Title"
author: peng
categories: [AI & ML]
tags: [draft-tag]
---
```

**Preview Drafts Locally:**
```bash
localhost --draft
# Visit http://127.0.0.1:4000/
```

**Publishing Drafts:**
1. Add date to frontmatter: `date: 2026-01-27`
2. Move file to `_posts/` with date prefix: `mv _drafts/my-draft-post.md _posts/2026-01-27-my-draft-post.md`
3. Commit and push to trigger deployment

**Note:** Drafts are **not** deployed to production (GitHub Pages ignores `_drafts/` directory).

### Header Images

**Location:** `assets/headers/`

**Naming Convention:** `YYYY-MM-DD-post-title.webp`

**Format:** WebP preferred for performance (smaller file size, good quality)

**Dimensions:** Should be consistent with existing headers (typically 1200x630 or similar)

**Usage in Frontmatter:**
```yaml
image:
  path: assets/headers/2026-01-27-my-post.webp
  alt: "Descriptive text for screen readers"
```

## Git Workflow

### Commit Message Conventions

Follow **conventional commits** format for all commits:

**Format:** `<type>(<scope>): <description>`

**Common Types:**
- `feat(blog):` - New blog post
- `refactor(blog):` - Content refactoring or updates to existing post
- `fix(blog):` - Bug fixes in content or layout
- `chore(config):` - Configuration changes
- `docs:` - Documentation updates (including this CLAUDE.md)
- `style:` - Formatting, no content change
- `test:` - Testing updates

**Examples:**
```bash
feat(blog): add MLflow structured learning tutorial
feat(blog): add GitHub Actions race condition debugging post
refactor(blog): rename category Intelligence & ML to AI & ML
fix(blog): correct typo in beta distribution post
chore(config): clean up Jekyll configuration and add blog header image
docs: update CLAUDE.md with category system rules
```

**Commit Message Rules:**
- Use imperative mood ("add" not "added", "fix" not "fixed")
- Keep subject line under 72 characters
- No period at end of subject line
- Capitalize first letter of description
- **DO NOT** add Claude as co-author in commits

### Branch Workflow

**Feature Branch Workflow:**
```bash
# Create feature branch
git checkout -b feat/new-blog-post

# Make changes and commit
git add _posts/2026-01-27-new-post.md assets/headers/2026-01-27-new-post.webp
git commit -m "feat(blog): add new post on topic"

# Push to remote
git push -u origin feat/new-blog-post

# Create pull request to main branch
gh pr create --title "feat(blog): add new post on topic" --body "Description of changes"
```

**Direct to Main (for minor changes):**
```bash
# For small fixes or updates, can push directly to main
git checkout main
git add <files>
git commit -m "fix(blog): correct typo"
git push origin main
```

### Staging Files

**Prefer specific file staging over `git add .`:**
```bash
# Good: Stage specific files
git add _posts/2026-01-27-new-post.md
git add assets/headers/2026-01-27-new-post.webp

# Avoid: Staging all files (may include unintended files)
git add .
git add -A
```

**Rationale:** Prevents accidentally committing:
- Sensitive files (.env, credentials)
- Build artifacts (_site/, .jekyll-cache/)
- Editor temporary files

## Testing and Verification

### Pre-Commit Checklist

Before committing changes:
- [ ] Site builds without errors
- [ ] Post appears in correct category page
- [ ] Header image displays correctly
- [ ] No broken internal links
- [ ] Category structure is flat (single category only)
- [ ] Tags are appropriate and consistent with existing posts
- [ ] Math rendering works (if `math: true`)
- [ ] TOC is generated properly

### Local Testing

**Build Site:**
```bash
bundle exec jekyll build
# Check _site/ directory for generated output
```

**Test with html-proofer:**
```bash
./tools/test.sh
# Runs build + html-proofer for link validation
```

**Preview in Browser:**
```bash
localhost --draft
# Visit http://127.0.0.1:4000/
# Check:
# - Post appears on home page
# - Category page shows post
# - Images load correctly
# - Links work
# - Code highlighting works
# - Math rendering works (if applicable)
```

### Common Issues

**Port Already in Use:**
```bash
# Solution 1: Use localhost command (auto-kills conflicts)
localhost

# Solution 2: Manual kill
lsof -ti :4000 | xargs kill -9

# Solution 3: Use different port
localhost --port 4001
```

**Category Nested Folders:**
```
Error: Post appears at /categories/AI/ML/ instead of /categories/AI-ML/
Cause: Multiple categories in array format
Solution: Use single category only, move secondary topics to tags
```

**Site Not Building:**
```bash
# Check Ruby version (should be 3.3+)
ruby -v

# Reinstall gems
bundle install

# Clear cache
rm -rf _site .jekyll-cache

# Rebuild
bundle exec jekyll build
```

**Images Not Loading:**
```
Check:
- File path is correct (assets/headers/filename.webp)
- File exists in repository
- Path in frontmatter matches actual file location
- File is committed to git
```

## CI/CD Pipeline

### GitHub Actions Workflow

**Workflow File:** `.github/workflows/pages-deploy.yml`

**Trigger Conditions:**
- Push to `main` or `master` branch
- Pull request to `main` or `master` (build only, no deploy)
- Manual trigger via Actions tab (workflow_dispatch)

**Build Process:**
1. Checkout repository with full git history
2. Setup GitHub Pages configuration
3. Setup Ruby 3.3 with bundler cache
4. Build site with Jekyll (production environment)
5. Test site with html-proofer (external links disabled)
6. Upload site artifact

**Deployment Process:**
- Only runs on push to main/master (not on PRs)
- Deploys to GitHub Pages environment
- Concurrent deployment prevention enabled (no race conditions)

**Link Validation:**
- html-proofer checks internal links
- External links disabled (`--disable-external`) to prevent false failures
- Ignores localhost URLs during testing

### Monitoring Deployments

**View Workflow Runs:**
```bash
# Using GitHub CLI
gh run list

# View specific run
gh run view <run-id>

# Watch latest run
gh run watch
```

**Check Deployment Status:**
- Visit: https://github.com/pengguanya/pengguanya.github.io/actions
- Look for "Build and Deploy" workflow
- Green checkmark = successful deployment
- Red X = failed, click for details

**Common Workflow Failures:**
- Broken internal links (html-proofer catches these)
- Invalid frontmatter YAML syntax
- Missing dependencies in Gemfile
- Ruby version mismatch

### Deployment Timing

**After Pushing to Main:**
1. GitHub Actions workflow starts automatically (within seconds)
2. Build and test phase (~2-3 minutes)
3. Deployment phase (~1 minute)
4. Site propagation (instant to few minutes)
5. **Total time:** ~5 minutes from push to live

**Pull Requests:**
- Build and test run, but no deployment
- Ensures changes don't break the site
- Review deployment preview in Actions tab

## Site Configuration

### Main Configuration File

**File:** `_config.yml`

**Key Settings:**
- `title`: Site title (appears in header)
- `tagline`: Site subtitle
- `description`: SEO description
- `url`: Full site URL (https://pengguanya.github.io)
- `timezone`: Site timezone (currently empty, defaults to UTC)
- `paginate`: Posts per page (10)
- `toc`: Global TOC setting (true)
- `comments`: Giscus integration for comments

**Comment System:**
- Provider: Giscus (GitHub Discussions)
- Repository: pengguanya/pengguanya.github.io
- Enabled on all posts by default
- Can disable per post with `comments: false` in frontmatter

### Author Configuration

**File:** `_data/authors.yml`

Define author information used in posts:
```yaml
peng:
  name: Guanya Peng
  twitter: <username>
  url: https://pengguanya.github.io
```

Reference in post frontmatter: `author: peng`

### Social Links

**File:** `_data/contact.yml`

Configure social media links shown in sidebar.

### Jekyll Archives

**Plugin:** jekyll-archives (enabled in _config.yml)

**Auto-Generates:**
- Category pages: `/categories/<category-name>/`
- Tag pages: `/tags/<tag-name>/`

**Configuration:**
```yaml
jekyll-archives:
  enabled: [categories, tags]
  layouts:
    category: category
    tag: tag
  permalinks:
    tag: /tags/:name/
    category: /categories/:name/
```

**No Manual Setup Required:** Jekyll auto-discovers categories and tags from post frontmatter.

## Chirpy Theme

### Theme Information

**Theme:** jekyll-theme-chirpy (~v7.1)

**Official Documentation:** https://github.com/cotes2020/chirpy-starter

**Key Features:**
- Responsive design (mobile-friendly)
- Light/dark mode toggle
- Built-in search
- TOC (Table of Contents) in posts
- Category and tag pages
- Archives page
- Syntax highlighting (Rouge)
- Math support (MathJax)
- Comments (Giscus)
- PWA support (installable as app)
- Feed/RSS
- SEO optimization

### Theme Customization

**Customized Files:**
- `_tabs/about.md`: Custom about page with blog pillars
- `_data/contact.yml`: Social links configuration
- `_config.yml`: Site-specific settings

**Layout Files:**
Theme layouts inherited from gem, not in repository (standard Jekyll gem-based theme approach).

### Math Support

**Enable Math in Post:**
```yaml
math: true
```

**Usage in Markdown:**
```markdown
Inline math: $E = mc^2$

Block math:
$$
\int_{a}^{b} f(x) dx
$$
```

**Rendering:** MathJax library loaded when `math: true` in frontmatter.

### Code Highlighting

**Automatic:** Rouge syntax highlighter (configured in _config.yml)

**Usage:**
````markdown
```python
def hello_world():
    print("Hello, World!")
```
````

**Supported Languages:** All languages supported by Rouge (Python, R, JavaScript, Bash, etc.)

## Common Tasks

### Adding a New Blog Post

```bash
# 1. Create post file
touch _posts/2026-01-27-my-topic.md

# 2. Add frontmatter and content
cat > _posts/2026-01-27-my-topic.md << 'EOF'
---
title: "My Post Title"
date: 2026-01-27
author: peng
categories: [AI & ML]
tags: [tag1, tag2]
math: false
image:
  path: assets/headers/2026-01-27-my-topic.webp
  alt: "Image description"
---

Post content here...
EOF

# 3. Add header image (if applicable)
cp /path/to/image.webp assets/headers/2026-01-27-my-topic.webp

# 4. Preview locally
localhost --draft

# 5. Commit and push
git add _posts/2026-01-27-my-topic.md assets/headers/2026-01-27-my-topic.webp
git commit -m "feat(blog): add post on my topic"
git push origin main
```

### Updating an Existing Post

```bash
# 1. Edit post file
vi _posts/2026-01-27-existing-post.md

# 2. Preview changes
localhost

# 3. Commit with appropriate type
git add _posts/2026-01-27-existing-post.md
git commit -m "refactor(blog): update existing post with new information"
git push origin main
```

### Renaming a Category

**Important:** Renaming categories affects multiple posts.

```bash
# 1. Find all posts with old category
grep -r "categories:.*Old Category" _posts/

# 2. Update each post's frontmatter
# Use Edit tool or sed to replace category name

# 3. Commit all changes together
git add _posts/*.md
git commit -m "refactor(blog): rename category 'Old Category' to 'New Category'"
git push origin main
```

**Note:** Jekyll auto-regenerates category pages; no other changes needed.

### Adding New Tags

**No Setup Required:** Just add tags to post frontmatter.

```yaml
tags: [new-tag, another-tag]
```

Jekyll auto-discovers tags and generates tag pages via jekyll-archives plugin.

### Updating Site Configuration

```bash
# 1. Edit configuration
vi _config.yml

# 2. Restart local server (required for config changes)
# Stop current server (Ctrl+C)
localhost

# 3. Commit changes
git add _config.yml
git commit -m "chore(config): update site configuration"
git push origin main
```

**Note:** Configuration changes require server restart (Jekyll loads _config.yml at startup only).

## Development Tips

### Draft Workflow

**Recommended Approach:**
1. Create draft in `_drafts/` (no date prefix)
2. Preview with `localhost --draft`
3. Iterate on content
4. When ready: add date, move to `_posts/`, rename with date prefix
5. Commit and push

**Advantage:** Drafts don't clutter `_posts/` directory and won't accidentally get deployed.

### Using VS Code

**Extensions to Consider:**
- Markdown All in One
- Code Spell Checker
- YAML (for frontmatter)

**Tasks:**
- "Run Jekyll Server" task configured in `.vscode/tasks.json`
- Use Ctrl+Shift+B (Cmd+Shift+B on Mac) to run

### Performance Optimization

**Header Images:**
- Use WebP format (smaller size, good quality)
- Optimize images before adding to repository
- Typical size: 1200x630 pixels
- Keep file size under 200KB if possible

**Build Time:**
- Clean cache periodically: `rm -rf .jekyll-cache`
- Incremental builds enabled by default in Jekyll

### Content Guidelines

**Writing Style:**
- Technical but accessible
- Clear explanations of complex topics
- Code examples where applicable
- Include references and links to resources

**Post Length:**
- No strict limit, but aim for depth over breadth
- Typical: 1000-3000 words for technical tutorials
- Shorter posts acceptable for quick tips or updates

**Code Examples:**
- Use syntax highlighting with language tag
- Include comments for clarity
- Test code examples before publishing

## Troubleshooting

### Build Errors

**Error: Could not find gem 'jekyll-theme-chirpy'**
```bash
# Solution: Install dependencies
bundle install
```

**Error: Configuration file not found**
```bash
# Ensure you're in repository root
cd /home/pengg3/work/pengguanya.github.io
```

**Error: Invalid YAML frontmatter**
```
Check:
- YAML syntax is correct (colons, indentation)
- Arrays use brackets: [item1, item2]
- Strings with special chars are quoted
- No tabs (use spaces)
```

### Rendering Issues

**Math Not Rendering:**
```
Check:
- `math: true` in frontmatter
- Math delimiters: $...$ for inline, $$...$$ for block
- No spaces after opening $ or before closing $
```

**Code Not Highlighting:**
```
Check:
- Language specified in code fence: ```python
- Language name is valid (lowercase, no spaces)
```

**TOC Not Showing:**
```
Check:
- Post has headings (## or ###)
- `toc: true` in frontmatter or globally in _config.yml
- Headings use proper markdown syntax (# with space)
```

### Git Issues

**Merge Conflicts:**
```bash
# View conflicted files
git status

# Resolve conflicts in editor
vi <conflicted-file>

# Mark as resolved
git add <conflicted-file>

# Complete merge
git commit
```

**Accidentally Committed to Wrong Branch:**
```bash
# Save changes to new branch
git branch feat/saved-changes

# Reset current branch
git reset --hard origin/main

# Switch to new branch
git checkout feat/saved-changes
```

## Additional Resources

### Documentation

- **Jekyll Official Docs:** https://jekyllrb.com/docs/
- **Chirpy Theme:** https://github.com/cotes2020/jekyll-theme-chirpy
- **GitHub Pages:** https://docs.github.com/en/pages
- **Markdown Guide:** https://www.markdownguide.org/
- **Liquid Template Language:** https://shopify.github.io/liquid/

### Related Configuration

- **Parent Environment:** See `/home/pengg3/CLAUDE.md` for overall system setup
- **Work Directory:** See `/home/pengg3/work/CLAUDE.md` for project context

### Useful Commands

```bash
# Start local server
localhost [--draft] [--port PORT]

# Build site
bundle exec jekyll build

# Test site
./tools/test.sh

# Create new post (manual)
touch _posts/$(date +%Y-%m-%d)-title.md

# Find posts by category
grep -l "categories:.*AI & ML" _posts/*.md

# Count posts
ls -1 _posts/*.md | wc -l

# View git log with graph
git log --oneline --graph --all

# Check GitHub Actions status
gh run list
```

## Notes

- **No Claude Co-Author:** Do not add Claude as co-author in git commits
- **Category System:** Single category only - this is a hard rule to maintain flat structure
- **Conventional Commits:** Follow format consistently for clean git history
- **Preview Before Push:** Always test locally with `localhost` command before deploying
- **Header Images:** Optional but recommended for better visual appeal and SEO

## Questions or Issues?

If you encounter issues not covered in this guide:
1. Check Jekyll build output for specific error messages
2. Review GitHub Actions workflow logs in Actions tab
3. Consult Chirpy theme documentation for theme-specific issues
4. Check Jekyll official documentation for general Jekyll issues

Remember: The `localhost` command is your friend for local development - it handles port conflicts automatically and makes testing drafts easy!
