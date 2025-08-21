# 🧙‍♂️ Godot CI/CD GitHub Actions 🧙‍♂️

## Things it can do now

- Find, download, and install specific engine versions based on the official godot github repo's git tags
- Export project to all platforms defined in project exports
- Upload build artifacts to itch.io
- Stamp version numbers into the project.godot file (this is then available to GD script)
- Update the main scene configuration in project.godot files (Easily switch between different main scenes for win/linux/web or Change the splash screen! Custom configs too!)

## Things it can do in the future

- Upload to Steam
- Compile the engine from source (C++ plugins/extensions or whatever else...)
- Find, download, and install blender for blender imports

## Under Construction

- This project is under active early development and subject to change. Thanks to github's action versioning with tags, pinned versions of actions will not change, but may become unsupported quickly

# 🚀 Quick Start 🚀

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
  - uses: digiur/GDCICD/actions/fetch-engine@v0.18
  - uses: digiur/GDCICD/actions/build-project@v0.18
  - uses: digiur/GDCICD/actions/publish-itchio@v0.18
        with:
          itchio_target: "<ICHIO_USER>/<ICHIO_PROJECT>:web"
          api_key: ${{ secrets.ITCHIO_API_KEY }}
```

### 1. Define an HTML5 export preset named 'web' in your project

- In the editor go to **project > export**
- Create a new HTML5 export named 'web'
- More info [here](https://docs.godotengine.org/en/latest/tutorials/export/exporting_for_web.html)

### 2. Add a file `.github/workflows/gdcicd.yml` to your github repo and put the above snippit into it

- See `.github/workflows/itchio_example.yml` and the individual `action.yml` files in `./actions` for more detailed information about actions and inputs

### 3. Change `<ICHIO_USER>/<ICHIO_PROJECT>` to your itchio username and project name

These can easily be found in the URL of your game's itch.io page.

- For this URL: `https://digiur.itch.io/my-game-project`
- You would use: `digiur/my-game-project:web`
- More info on itch.io's butler [here](https://itch.io/docs/butler/pushing.html)

### 4. Add Your Itch.io API Key

- Go to your repo’s **Settings > Secrets and variables > Actions > New repository secret**
- Name it: `ITCHIO_API_KEY`
- Paste your [Itch.io API key](https://itch.io/user/settings/api-keys)
- More info on github secrets [here](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)

### 5. Run the action!

- Go to your repo's Actions tab and select your new 'Godot Build and Publish to Itch.io' action in the left column
- You should see a message 'This workflow has a `workflow_dispatch` event trigger.' and a 'Run Workflow' button to run the workflow

### 🪄 Your project will build and deploy to itchio! 🪄

- Any errors or warnings will be printed to the action's output with suggestions for fixes
- Open an issue or start a discussion on this repo if you need help!

# 🦄 Upgrade Options! 🦄

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

The edit-config action can edit any key in your project.godot file using section/key/value inputs.

```yaml
- uses: digiur/GDCICD/actions/edit-config@v0.18
  with:
    section: application
    key: config/version
    value: "1.2.3-153"
```

This will set `config/version="1.2.3-153"` in your project.godot file. The version number is available to gdscript at run time so you can display `Version: 1.2.3-153` somewhere in-game.

### GitVersion

[GitVersion](https://github.com/marketplace/actions/gittools) is a tool to generate semantic version numbers for your build. It can be combined with the `edit-config` action to automatically increment the version numbers of your build. It has many [outputs](https://github.com/GitTools/actions/blob/main/docs/examples/github/gitversion/execute.md#outputs) to choose from.

```yaml
steps:
  - uses: gittools/actions/gitversion/setup@v4.1.0 # install gitversion
    with:
      versionSpec: "6.3.x"

  - uses: gittools/actions/gitversion/execute@v4.1.0 # run gitversion

  - uses: digiur/GDCICD/actions/edit-config@v0.18
    with:
      section: application
      key: config/version
      value: ${{ env.semVer }} # gitversion sets a number of env vars to choose from
```

## Exporting with Different Main Scenes for Different Platforms

You can use the `edit-config` action to set a different main scene before each export. For example, to export a Windows build with one main scene and a Web build with another:

```yaml
jobs:
  build-multi-platform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Set main scene for Windows export
      - uses: digiur/GDCICD/actions/edit-config@v0.18
        with:
          section: application
          key: run/main_scene
          value: "res://windows_main.tscn"

      - uses: digiur/GDCICD/actions/build-project@v0.18
        with:
          export_target: windows
          export_path: builds/windows
          export_file: game.exe

      # Set main scene for Web export
      - uses: digiur/GDCICD/actions/edit-config@v0.18
        with:
          section: application
          key: run/main_scene
          value: "res://web_main.tscn"

      - uses: digiur/GDCICD/actions/build-project@v0.18
        with:
          export_target: web
          export_path: builds/web
          export_file: index.html
```

## Save Build Artifacts to GitHub

- The GitHub platform provaides an action for that! 'upload-artifact'

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

  - uses: digiur/GDCICD/actions/fetch-engine@v0.18

  - uses: actions/upload-artifact@v4
      with:
        name: after_engine_fetch
        path: .
        retention-days: 1
        overwrite: true
        include-hidden-files: true

  - uses: digiur/GDCICD/actions/build-project@v0.18

  - uses: actions/upload-artifact@v4
      with:
        name: after_build
        path: .
        retention-days: 1
        overwrite: true
        include-hidden-files: true
```

## 🛟 Need Help? 🛟

Open an issue or discussion in this repo!
