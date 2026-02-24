# 0. Pre-req:
[uv](https://docs.astral.sh/uv/)

# 1. Setup & Initialize MkDocs
```bash
# 1. Create and enter a new directory
mkdir my-docs # OR create repo in Github
cd my-docs

# 2. Install mkdocs with material design
uv tool install mkdocs --with mkdocs-material

# 3. Add the theme configuration to mkdocs.yml
# (This command appends the required theme block directly to the file)
echo -e "theme:\n  name: material" >> mkdocs.yml

# 6. Start the local development server
mkdocs serve
```

# 2. Update MkDocs
```bash
uv tool upgrade mkdocs
uv tool install mkdocs --with mkdocs-material
uv tool upgrade --all
```

# 3. Delete MkDocs
```bash
uv tool uninstall mkdocs --with mkdocs-material
```

# 4. Deploy to GitHub Pages or Cloudflare Pages
## Option 1: Deploy to GitHub Pages (via GitHub Actions)
GitHub Pages is the most common way to host MkDocs sites. We will use a GitHub Action to automatically build and deploy your site every time you push to the main branch.

### 1. Create the workflow file
In your project directory, create a .github/workflows folder and add a file named deploy.yml:

```bash
mkdir -p .github/workflows
touch .github/workflows/deploy.yml
```

### 2. Add the Action configuration
Copy and paste the following into deploy.yml. This uses Astral's official setup-uv action for lightning-fast dependency installation:

```yaml
name: deploy
on:
  push:
    branches:
      - main
      - master
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
          
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          
      - name: Install dependencies
        run: uv sync
        
      - name: Deploy to GitHub Pages
        run: uv run mkdocs gh-deploy --force
```
### 3. Push to GitHub
Commit your code and push it to GitHub. MkDocs will automatically create a gh-pages branch and publish your site. (Note: You may need to go to your repository Settings > Pages and ensure the source is set to the gh-pages branch).

## Option 2: Deploy to Cloudflare Pages
Cloudflare Pages is incredibly fast and connects directly to your GitHub repository without needing a workflow file in your code.

### 1. Push your code to GitHub
Make sure your pyproject.toml, uv.lock, and mkdocs.yml are pushed to your GitHub repository.

### 2. Connect Cloudflare to GitHub

Log in to your Cloudflare Dashboard.

Navigate to Workers & Pages > Create application > Pages > Connect to Git.

Select your repository and click Begin setup.

### 3. Configure the Build Settings
Cloudflare needs to know how to install uv and build your site. Fill out the build settings exactly like this:

Framework preset: None

Build command: pip install uv && uv sync && uv run mkdocs build

Build output directory: site

### 4. Set Environment Variables
Cloudflare Pages sometimes defaults to older Python versions. To ensure uv and MkDocs run properly, add an environment variable:

Click Environment variables (advanced).

Add a variable named PYTHON_VERSION and set its value to 3.11.

### 5. Save and Deploy
Click Save and Deploy. Cloudflare will pull your code, install uv, sync your lockfile, and publish your site globally. Every time you push to your main branch, Cloudflare will automatically rebuild it.
