# Contributing to Browseraptor Channel

Thank you for your interest in contributing to the Browseraptor plugin ecosystem. This guide will help you submit your plugin to the default channel.

## Review Existing Plugins

Start by looking for similar plugins you might be able to contribute to.

We strongly encourage improving or adding to existing plugins. This means that plugins are of higher quality, and users don't have to choose from several similar options. We regularly replace plugins that have become outdated with newer implementations.

## Pick a Name

Beyond taste there are some technical aspects to keep in mind when naming your plugin:

- Try not to use the word "Browseraptor" in your plugin name. Every plugin available via Browseraptor is for Browseraptor. You can use the word Browseraptor in your marketing material (README etc.) though.
- Don't use a name too similar to another: we don't want `Todo` and `T0d0`.
- Don't use snake_case. It's uncommon and looks weird.
- Don't use a `.` in the package name. If your plugin includes any Rust code, it may cause issues with module loading.
- Don't use a `/` or other restricted characters in the package name. Invalid characters include: `<`, `>`, `:`, `"`, `/`, `\`, `|`, `?` and `*`.
- Use ASCII only.
- Language support (aka "syntax" or "grammar") packages are named after the language it supports, without suffixes like "syntax" or "highlighting".

Note that the package name is used in references to resources. If your package name is different from its Git repository name, make sure you rename the local clone to match.

## Host the Plugin

1. **A public GitHub repository**
   - Only include one plugin per Git repository
   - Be sure the root of the plugin is the root of the repository
   - Do not include a `repository.json` file in your repository
   - You will need to create a tag each time you want to make a new version available to users. The tag names must be a semantic version number.

## Prepare Your Git Repository

- Create a semver-numbered tag (e.g., `v1.0.0`)
- Ensure there are no compiled artifacts in your repository (e.g., `target/` directory for Rust projects)
- Remove any auto-generated metadata files
- Check file names for cross-platform compatibility (same restricted characters apply as in your package name)
- Under very specific circumstances, like including executables or shared libraries, add a `.no-browseraptor-plugin` file to the root of your repository. This file will prevent Browseraptor from shipping your plugin as a zipped file.

## Browseraptor Plugin API

Browseraptor plugins are compiled to WebAssembly (WASM) and must export specific functions to interface with the host.

### Required WASM Exports

Your plugin must export the following functions:

```rust
// Allocate memory in the WASM module
#[no_mangle]
pub extern "C" fn alloc(size: i32) -> i32;

// Deallocate memory in the WASM module
#[no_mangle]
pub extern "C" fn dealloc(ptr: i32, size: i32);

// Evaluate a URL and return a result
// Input: (ptr: i32, len: i32) - pointer and length of URL string
// Output: i32 - pointer to result (4 bytes length + JSON data)
#[no_mangle]
pub extern "C" fn evaluate(ptr: i32, len: i32) -> i32;
```

Your plugin must also export a `memory` export (standard WASM memory).

### Plugin Result Format

The `evaluate` function must return a pointer to a JSON-serialized `PluginResult`:

```rust
pub struct PluginResult {
    pub browser: Option<String>,  // Browser name to use (e.g., "firefox", "chrome")
    pub profile: Option<String>,   // Browser profile to use (e.g., "work", "personal")
    pub cancel: bool,              // If true, cancel the navigation entirely
}
```

**Result format in memory:**

- First 4 bytes: result length as little-endian i32
- Following bytes: JSON-serialized `PluginResult`

### Example Plugin (Rust)

```rust
use serde::{Deserialize, Serialize};
use std::ffi::CStr;
use std::os::raw::c_char;

#[derive(Serialize, Deserialize)]
pub struct PluginResult {
    pub browser: Option<String>,
    pub profile: Option<String>,
    pub cancel: bool,
}

#[no_mangle]
pub extern "C" fn alloc(size: i32) -> i32 {
    // Allocate memory (implementation depends on your WASM runtime)
    // Return pointer to allocated memory
    0 // placeholder
}

#[no_mangle]
pub extern "C" fn dealloc(ptr: i32, size: i32) {
    // Deallocate memory
}

#[no_mangle]
pub extern "C" fn evaluate(ptr: i32, len: i32) -> i32 {
    // Read URL from memory at ptr with length len
    // Process URL and determine action
    // Return pointer to JSON-serialized PluginResult
    
    let result = PluginResult {
        browser: Some("firefox".to_string()),
        profile: Some("work".to_string()),
        cancel: false,
    };
    
    let json = serde_json::to_vec(&result).unwrap();
    // Write length + json to memory and return pointer
    0 // placeholder
}
```

### Plugin Manifest

Your plugin metadata file must include a manifest with:

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "author": "Your Name",
  "description": "Description of what the plugin does"
}
```

## Add Your Plugin to the Default Channel

1. Fork the [Browseraptor Channel](https://github.com/livrasand/browseraptor_channel) repository.
2. Add your plugin entry to `repository/index.json` in alphabetical order by plugin ID.
3. Create a plugin metadata file in `repository/{first-letter}/{plugin-id}.json` (e.g., `repository/m/my-plugin.json`).

### Plugin Metadata File Format

Your metadata file must follow this structure:

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "description": "Brief description of what your plugin does",
  "author": {
    "name": "Your Name",
    "url": "https://github.com/yourusername"
  },
  "version": "1.0.0",
  "license": "MIT",
  "categories": ["url-routing", "productivity"],
  "tags": ["github", "developer", "routing"],
  "min_browseraptor_version": "0.2.0",
  "permissions": ["navigation"],
  "wasm": {
    "url": "https://github.com/yourusername/my-plugin/releases/download/v1.0.0/plugin.wasm",
    "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "size_bytes": 24576,
    "entry_point": "evaluate",
    "api_version": "1.0.0"
  },
  "repository": {
    "url": "https://github.com/yourusername/my-plugin",
    "type": "git"
  },
  "changelog": "https://github.com/yourusername/my-plugin/releases",
  "docs": "https://github.com/yourusername/my-plugin#readme",
  "updated_at": "2026-06-06T00:00:00Z",
  "created_at": "2026-06-06T00:00:00Z"
}
```

**Important:**

- The `wasm.url` must point to a release asset in your GitHub repository
- The `wasm.sha256` must be the SHA256 hash of the WASM file
- The `id` must match the filename (without `.json`)
- The file must be placed in the directory corresponding to the first letter of the ID

## Submit a Pull Request

Now you're ready to push your changes and make a PR on the Browseraptor Channel repository. Follow the guidelines in the [PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md) and make sure the tests pass!

For a cleaner process, you should only submit one plugin per PR. This allows plugin submissions to be reviewed and merged independently.

Note that this is a community project and people review PRs in their spare time; it might take a while.

## Things That Help Your Submission Get Approved More Quickly

- We only accept submissions from maintainers of the plugin being submitted.
- A valid semver numbered tag must exist on the repository.
- Ensure the README clearly describes the purpose of the plugin and how to use it.
- Your plugin must compile to a valid WASM file that exports the required functions.
- The WASM file should be reasonably sized (preferably under 1MB).
- Include proper error handling in your plugin code.
- Test your plugin with various URL patterns before submitting.
- Use descriptive categories and tags to help users discover your plugin.

## Categories and Tags

Use categories and tags to help users discover your plugin:

**Categories** (max 3, choose from):

- `url-routing` - Plugins that route URLs based on patterns
- `automation` - Plugins that automate browser actions
- `productivity` - Plugins that improve workflow
- `developer-tools` - Plugins for developers
- `communication` - Plugins for communication platforms
- `forms` - Plugins that handle form interactions

**Tags** (max 10, be specific):

- Use lowercase tags
- Examples: `github`, `jira`, `slack`, `login`, `auth`, `redirect`, `work`, `personal`

## A Word About AI Use

Although we use automation, a human will be reviewing your submission. An important part of this is an assessment of the ability and willingness of the human submitting the plugin (i.e. you) to support it long term. This is therefore primarily a human to human conversation. While you're welcome to use any form of automation, you are expected to participate in this conversation yourself.

## Questions?

If you have questions about plugin development or the submission process, please open an issue in the [Browseraptor Channel](https://github.com/livrasand/browseraptor_channel) repository.
