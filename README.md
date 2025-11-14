# wasmagic
auto-wasm conversion
Mwahaha! You have struck upon a glorious idea: a **generalized GitHub Action** that automates the transmutation of **multiple repositories into WebAssembly artifacts**, all orchestrated from a central repository. Let us build this majestic "Wasm Magic Factory" (`.wasmagic`) together, where repositories are listed, processed, and converted in a seamless, automaton-driven workflow! 🧪✨

---

### **The Vision in Detail**
1. **Folder Structure**:
   - The central repo (`Wasm Magic Factory`) contains a special folder: `.wasmagic/`.
   - Inside `.wasmagic/`:
     - A `convert-me` file lists the URLs of repositories to be converted.
     - Generated Wasm artifacts for each target repo are stored in a new folder named `{{target_repo}}_wasm`.

2. **Workflow Steps**:
   - Clone each target repository listed in `convert-me` into a folder named `{{target_repo}}`.
   - Compile the target repository to Wasm, saving the results in `{{target_repo}}_wasm`.
   - Process all repositories in one glorious batch!

---

### **Folder Structure of Central Repo**
Here’s what the **Wasm Magic Factory** repository will look like:

```
/
|-- .github/
|   |-- workflows/
|       |-- wasmagic.yml          # The GitHub Action workflow
|-- .wasmagic/
|   |-- convert-me                # File containing the URLs of target repositories
|-- README.md
```

---

### **The Workflow: Automate the Magic**

Save the following workflow file as `.github/workflows/wasmagic.yml` in the central repository:

```yaml
name: Wasm Magic Factory

on:
  workflow_dispatch: # Allow manual trigger of this workflow

jobs:
  convert-repos-to-wasm:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Checkout THIS repository (Wasm Magic Factory)
      - name: Checkout This Repository
        uses: actions/checkout@v4

      # Step 2: Read the 'convert-me' File and Process Each Repository
      - name: Read Target Repositories
        run: |
          echo "Reading repositories from .wasmagic/convert-me..."
          
          # Read the URLs from the file
          while IFS= read -r repo_url || [[ -n "$repo_url" ]]; do
            echo "Processing repository: $repo_url"

            # Extract repository name (e.g., 'example-repo' from 'https://github.com/user/example-repo.git')
            repo_name=$(basename -s .git "$repo_url")
            echo "Repository name: $repo_name"

            # Clone the repository into {{target_repo}}
            echo "Cloning $repo_url into $repo_name..."
            git clone --depth=1 "$repo_url" "$repo_name"

            # Create an output folder for Wasm artifacts: {{target_repo}}_wasm
            mkdir -p "${repo_name}_wasm"

            # Compile the repository to Wasm
            echo "Compiling $repo_name to WebAssembly..."
            if [ -f "$repo_name/Cargo.toml" ]; then
              # Rust project
              echo "Detected Rust project. Compiling..."
              cd "$repo_name"
              rustup target add wasm32-unknown-unknown
              cargo build --release --target wasm32-unknown-unknown
              cd ..

              # Copy the compiled Wasm binary to the output folder
              cp "$repo_name/target/wasm32-unknown-unknown/release/"*.wasm "${repo_name}_wasm/"
            elif [ -f "$repo_name/Makefile" ] || [ -f "$repo_name/CMakeLists.txt" ]; then
              # C/C++ project
              echo "Detected C/C++ project. Compiling..."
              git clone https://github.com/emscripten-core/emsdk.git
              cd emsdk
              ./emsdk install latest
              ./emsdk activate latest
              source ./emsdk_env.sh
              cd ..

              cd "$repo_name"
              emmake make
              cd ..

              # Copy the generated Wasm to the output folder
              cp "$repo_name/build/"*.wasm "${repo_name}_wasm/"
            else
              echo "Unknown project type for $repo_name. Skipping..."
              continue
            fi

            echo "Compilation complete for $repo_name. Wasm artifacts stored in ${repo_name}_wasm."
          done < .wasmagic/convert-me

      # Step 3: Archive All Generated Wasm Artifacts
      - name: Upload All Wasm Artifacts
        uses: actions/upload-artifact@v3
        with:
          name: all-wasm-artifacts
          path: |
            *_wasm/
```

---

### **How It Works**

1. **Target Repositories**:
   - The `convert-me` file in the `.wasmagic/` folder lists the repositories to be converted. Each line is a URL for a repository to be processed.

   Example `convert-me` file:
   ```
   https://github.com/OzCog/opencog-central
   https://github.com/user/example-repo
   https://github.com/another-user/another-repo
   ```

2. **Steps in the Workflow**:
   - **Step 1: Clone Repositories**:
     - Each repository is cloned into a folder named `{{target_repo}}` (e.g., `opencog-central`).
   - **Step 2: Detect Language and Compile**:
     - For **Rust projects** (detected via `Cargo.toml`):
       - Compiles using `cargo build --target wasm32-unknown-unknown`.
     - For **C/C++ projects** (detected via `Makefile` or `CMakeLists.txt`):
       - Uses **Emscripten** to generate Wasm.
     - Skips unrecognized projects, emitting a warning.
   - **Step 3: Store Artifacts**:
     - The generated Wasm files are saved in a corresponding folder: `{{target_repo}}_wasm`.

3. **Output**:
   - At the end of the workflow, all `*_wasm` folders (containing Wasm artifacts) are archived as a downloadable artifact using `actions/upload-artifact`.

---

### **How to Use**
1. **Setup Your Central Repository**:
   - Create a repository that acts as the Wasm Magic Factory.
   - Add the `.wasmagic/convert-me` file and list the target repositories you want to convert.

2. **Run the Workflow**:
   - Go to the "Actions" tab in your central repository and trigger the `Wasm Magic Factory` workflow manually.

3. **Download Results**:
   - After the workflow finishes, download the `all-wasm-artifacts.zip` archive to retrieve the Wasm output for all target repositories.

---

### **Customizations**

- **Support More Languages**:
   Add custom build logic for other languages, such as Go, AssemblyScript, or Python.

- **Deploy Wasm Artifacts**:
   Automatically deploy generated Wasm files to a storage bucket, GitHub Pages, or another hosting service.

- **Notification**:
   Add a step to notify you (via Slack, email, etc.) when the workflow completes.

---

### **Final Thoughts**

This **generalized Wasm Factory** is perfect for automating the generation of WebAssembly across multiple repositories. Whether you're a mad scientist looking to dominate the web or just someone who hates manually compiling Wasm (how pedestrian!), this setup will save you time and effort.

Now go, build your `.wasmagic` empire! If you wish to extend it further, summon me, and I’ll unleash even greater automations! Mwahaha! 🧪✨

