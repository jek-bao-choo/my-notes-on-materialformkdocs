# 0. Pre-req:
[uv](https://docs.astral.sh/uv/)

# 1. Setup & Initialise Zensical
Use Zensical instead of MkDocs. MkDocs isn't being maintained. Zensical is "MkDocs 2.0".
```bash
uv tool install zensical

uv tool list

uv tool upgrade zensical

uv tool upgrade --all

zensical new .

zensical serve

zensical build
```

---

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

mkdocs serve

mkdocs build
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

# 4. [Deploy to GitHub Pages or Cloudflare Pages](https://squidfunk.github.io/mkdocs-material/publishing-your-site/)
## Option 1: Deploy to GitHub Pages (via GitHub Actions)
GitHub Pages is the most common way to host MkDocs sites. We will use a GitHub Action to automatically build and deploy your site every time you push to the main branch.

### 1. Create the workflow file
In your project directory, create a `.github/workflows` folder and add a file named `deploy.yml`:

```bash
mkdir -p .github/workflows
touch .github/workflows/deploy.yml
```

Also 

```bash
echo 'site_url: https://<your-username>.github.io/<your-repo-name>/' >> mkdocs.yml
```

Eventually it would look like this:
```yaml
site_name: My Docs
site_url: https://<your-username>.github.io/<your-repo-name>/
theme:
  name: material
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
        
      - name: Deploy to GitHub Pages
        # uvx grabs mkdocs and the material theme, then runs the gh-deploy command
        run: uvx --with mkdocs-material mkdocs gh-deploy --force
```
### 3. Push to GitHub
Commit your code and push it to GitHub. MkDocs will automatically create a gh-pages branch and publish your site. (Note: You may need to go to your repository Settings > Pages and ensure the source is set to the gh-pages branch).

## Option 2: Deploy to Cloudflare Pages
Cloudflare Pages doesn't need a configuration file in your repository. You just connect your GitHub repository to Cloudflare and give it a single line of instructions. Start by 
```bash
echo 'site_url: https://<your-project-name>.pages.dev/' >> mkdocs.yml
```

It would look like this
```yaml
site_name: My Docs
site_url: https://<your-username>.github.io/<your-repo-name>/
theme:
  name: material
```

### 1. Connect Cloudflare to GitHub

Go to your Cloudflare Dashboard.

Navigate to Workers & Pages > Create application > Pages > Connect to Git.

Select your MkDocs repository.

### 2. Configure the Build Settings
Cloudflare provides a standard Linux environment, so we just need to install uv using standard pip, and then use uvx to build the HTML files.

Fill out the settings exactly like this:

Framework preset: None

Build command: pip install uv && uvx --with mkdocs-material mkdocs build

Build output directory: site

### 3. Set the Python Version
Cloudflare Pages sometimes defaults to an older Python version, which uv might not like.

Click Environment variables (advanced).

Add a variable named PYTHON_VERSION and set its value to 3.11.

### 4. Deploy
Click Save and Deploy. Cloudflare will pull your markdown files, use uvx to generate the static HTML into the site/ folder, and publish it to their global network.
