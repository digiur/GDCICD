# GD-CICD: Godot CI/CD GitHub Actions

Easily automate building, exporting, and publishing your Godot projects with these reusable GitHub Actions!

## 🚀 Quick Start

### 1. **Add These Actions to Your Project**

Copy the `.github/workflows/itchio_example.yml` file from this repo to your own Godot project, or create a new workflow file (e.g., `.github/workflows/godot-ci.yml`).

### 2. **Set Up Your Export Presets**

The default itchio example requires a single HTML5 export prset named 'web' (Project -> Export -> Add...)

### 3. **Add Your Itch.io API Key**

- Go to your repo’s **Settings > Secrets and variables > Actions > New repository secret**
- Name it: `ITCHIO_API_KEY`
- Paste your [Itch.io API key](https://itch.io/user/settings/api-keys)

### 4. **Example Workflow**

Here’s a minimal workflow that fetches the engine, builds a project, and publishes to Itch.io:

```yaml
name: Godot Build and Publish to Itch.io

on:
  workflow_dispatch:

jobs:
  build-and-publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: digiur/GDCICD/actions/fetch-engine@v0.1

      - uses: digiur/GDCICD/actions/build-project@v0.1

      - uses: digiur/GDCICD/actions/publish-itchio@v0.1
        with:
          itchio_target: "YOUR_ITCHIO_USER/YOUR_PROJECT:web"
          butler_api_key: ${{ secrets.ITCHIO_API_KEY }}
```

- Change `itchio_target` to your Itch.io username and project.
- For other platforms, adjust the export preset and publishing step as needed.

## ❓ Need Help?

Open an issue or discussion in this repo!
