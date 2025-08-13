# Thinkmetal Usage Guide

## Prerequisites

### 1. Set Up Python Environment

It is recommended to use a virtual environment for Python tools:

- **Using `venv`:**

  ```sh
  python3 -m venv curaenv
  source curaenv/bin/activate
  ```

- **Or with `conda`:**

  ```sh
  conda create -n curaenv python=3.10
  conda activate curaenv
  ```

### 2. Install Conan

Install the required Conan version inside your environment:

```sh
pip install "conan==2.13.0"
```

### 3. Install and Source Emscripten SDK

To build for WASM, you need the Emscripten SDK. Follow these steps:

1. **Install Emscripten SDK:**

```sh
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
./emsdk install latest
./emsdk activate latest
```

2. **Source the Emscripten environment:**

```sh
source ./emsdk_env.sh
```

> **Note:**
> Follow the instructions and outputs provided by the Emscripten SDK (`emsdk`) during installation and activation. The SDK may prompt you with additional steps or guidance to ensure proper setup for your system.
> **Tip:**
> Add the above `source` command to your shell profile (e.g., `.bashrc` or `.zshrc`) for convenience.

## Conan Profile Setup(WASM)

1. **Detect your default build profile:**

   ```sh
   conan profile detect -f
   ```

   This creates `~/.conan2/profiles/default`.

2. **Create a host profile for Emscripten/WASM:**
   Create `~/.conan2/profiles/cura_wasm` with:

   ```ini
   include(cura.jinja)

   [tool_requires]
   emsdk/[>=3.1.73]@ultimaker/stable
   nodejs/[>=20.16.0]@ultimaker/stable

   [settings]
   os=Emscripten
   arch=wasm

   [conf]
   tools.build:skip_test=True
   tools.cmake.cmake_layout:build_folder_vars=['settings.os']
   user.curator:printers=['ultimaker_sketch','ultimaker_sketch_large','ultimaker_sketch_sprint','ultimaker_method','ultimaker_methodx','ultimaker_methodxl','ultimaker_s3']

   [options]
   curaengine/_:enable_plugins=False
   curaengine/_:enable_arcus=False
   curator/_:with_cura_resources=True
   curator/_:disable_logging=True
   dulcificum/_:with_python_bindings=False
   libzip/_:crypto=False
   ```

## Build Steps(in the repo folder)

1. **Clean previous build artifacts:**

   ```sh
   rm -rf build/emscripten/Release/*
   ```

2. **Install dependencies:**

   ```sh
   conan install . -pr:h=cura_wasm -pr:b=default --build=missing
   ```

3. **Configure the project:**

   ```sh
   cmake --preset conan-emscripten-release
   ```

4. **Build the project:**

   ```sh
   cmake --build --preset conan-emscripten-release
   ```

---

**Note:**
Always activate your Python environment and source the Emscripten environment before running any build commands.

## Keeping Your Fork Up To Date With Upstream

To keep your fork and your `main` branch synchronized with the upstream repository, follow these steps:

### 1. Add the Upstream Remote

If you haven't already, add the upstream repository as a remote:

```sh
git remote add upstream https://github.com/Ultimaker/CuraEngine.git
```

### 2. Fetch Upstream Changes

```sh
git fetch upstream
```

### 3. Update Your Local `main` Branch

Merge upstream changes into your local `main` branch:

```sh
git checkout main
git merge upstream/main
```

Or, for a linear history:

> **Warning:**  
> Rebasing can rewrite commit history. If you have local changes or branches, ensure they are backed up before rebasing. Attempting a rebase without understanding the process may result in lost work or conflicts. Only proceed if you are confident, and never rebase public/shared branches unless necessary.

```sh
git checkout main
git rebase upstream/main
```

### 4. Push Updated `main` to Your Fork

```sh
git push origin main
```

### 5. Sync `main` to `thinkmetal-internal-main`

To update your internal branch with the latest changes from `main`:

```sh
git checkout thinkmetal-internal-main
git merge main
```

Or, for a linear history:

> **Warning:**  
> Rebasing can rewrite commit history. If you have local changes or branches, ensure they are backed up before rebasing. Attempting a rebase without understanding the process may result in lost work or conflicts. Only proceed if you are confident, and never rebase public/shared branches unless necessary.

```sh
git checkout thinkmetal-internal-main
git rebase main
```

Then push the updated branch:

```sh
git push origin thinkmetal-internal-main
```

---

**Tip:**

- Always keep your internal branches (e.g., `thinkmetal-internal-main`) rebased or merged from your updated `main` to avoid conflicts.
- Never include internal-only folders (like `thinkmetal/`) in PRs to upstream unless intended.
