# Godot CI/CD GitHub Actions

## Under Construction

- This project is under active early development and subject to change. Thanks to github's action versioning with tags, pinned versions of actions will not change, but may become unsupported quickly

# 🚀 Quickest Start ⚡️

**Minimum possible workflow that fetches the engine, builds a project, and publishes to Itch.io**

```yaml
name: Godot Build and Publish to Itch.io
on:
  workflow_dispatch:
jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: digiur/GDCICD/actions/fetch-engine@v0.16
      - uses: digiur/GDCICD/actions/build-project@v0.16
      - uses: digiur/GDCICD/actions/publish-itchio@v0.16
        with:
          itchio_target: "<ICHIO_USER>/<ICHIO_PROJECT>:web"
          butler_api_key: ${{ secrets.ITCHIO_API_KEY }}
```

### 1. Add a file `.github/workflows/gdcicd.yml` to your github repo and put the above snippit into it

- See `.github/workflows/itchio_example.yml` and the individual `action.yml` files in `./actions` for more detailed information about actions and inputs

### 2. Change `<ICHIO_USER>/<ICHIO_PROJECT>` to your itchio username and project name

These can easily be found in the URL of your game's itch.io page.

- For this URL: `https://digiur.itch.io/my-game-project`
- You would use: `digiur/my-game-project:web`
- More info on itch.io's butler [here](https://itch.io/docs/butler/pushing.html)

### 3. Add Your Itch.io API Key

- Go to your repo’s **Settings > Secrets and variables > Actions > New repository secret**
- Name it: `ITCHIO_API_KEY`
- Paste your [Itch.io API key](https://itch.io/user/settings/api-keys)
- More info on github secrets [here](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)

### 4. Run the action!

- Go to your repo's Actions tab and select your new "Godot Build and Publish to Itch.io" action in the left column
- You should see a message "This workflow has a `workflow_dispatch` event trigger." and a "Run Workflow" button to run the workflow

### 🛠️ Your project will build and deploy to itchio! 🪄

- Any errors or warnings will be printed to the action's output with suggestions for fixes
- Open an issue or start a discussion on this repo if you need help!

# 🧙‍♂️ Upgrade Options! 🦄

## Run automatically

**Replace**

```yaml
on:
  workflow_dispatch:
```

**With**

```yaml
on:
  push:
    branches:
      - "main"
```

- The workflow will run automatically on every push to main. (Or any other branches you choose!)

## Automatic Versioning

**The stamp-version action will edit your project.godot file to add a semantic version to the build**

```yaml
- uses: digiur/GDCICD/actions/stamp-version@v0.16
  with:
    version: "1.2.3-153"
```

This version number is available to gdscript at run time so you can display `Version: 1.2.3-153` somewhere in-game.

### GitVersion

GitVersion is a tool to determine what version number a build should be. It can be combined with the `stamp-version` action to automatically increment the version numbers of your build.

```yaml
steps:
  - uses: gittools/actions/gitversion/setup@v4.1.0 # install gitversion
    with:
      versionSpec: '6.3.x'

- uses: gittools/actions/gitversion/execute@v4.1.0 # run gitversion

  - uses: digiur/GDCICD/actions/stamp-version@v0.16
    with:
      version: ${{ env.commitDate }} # gitversion sets a number of env vars to chose from
```

## Save Build Artifacts to GitHub

- The GitHub platform provaides an action for that! "upload-artifact"

```yaml
- uses: actions/upload-artifact@v4
    with:
      name: my_godot_build
      path: builds
      retention-days: 1
      overwrite: true
      include-hidden-files: false
```

- **name:** Name of the artifact to upload.
- **path:** A file, directory or wildcard pattern that describes what to upload. This should match the `export_path` used with the `digiur/GDCICD/actions/build-project` action Or `builds` if using the default `export_path`.
- **retention-days:** Duration after which artifact will expire in days. 0 means using default retention.
- **overwrite:** If true, an artifact with a matching name will be deleted before a new one is uploaded. If false, the action will fail if an artifact for the given name already exists.
- **include-hidden-files:** Whether to include hidden files in the provided path in the artifact.
- **Pro tips!**
  - Three different exports to `builds/win`, `builds/linux`, `builds/web`? Use a single artifact upload with `path: builds` to capture all three!
  - Keep retention low! If it expires, just run the build again...
  - Use `path: .` + `include-hidden-files: true` to see a snapshot of your project's file system for your build before, after, or in-between steps!
  - Use `path: /` to see a snapshot of the entire build system's filesystem! (Could be Very big!)

**The following example uploads 3 artifacts after_checkout, after_engine_fetch, and after_build. (This is for demonstration purposes, you probably don't want the first two artifacts in production...)**

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: actions/upload-artifact@v4
      with:
        name: after_checkout
        path: .
        retention-days: 1
        overwrite: true
        include-hidden-files: true

  - uses: digiur/GDCICD/actions/fetch-engine@v0.16

  - uses: actions/upload-artifact@v4
      with:
        name: after_engine_fetch
        path: .
        retention-days: 1
        overwrite: true
        include-hidden-files: true

  - uses: digiur/GDCICD/actions/build-project@v0.16

  - uses: actions/upload-artifact@v4
      with:
        name: after_build
        path: .
        retention-days: 1
        overwrite: true
        include-hidden-files: true
```

## 🛟 Need Help?

Open an issue or discussion in this repo!
