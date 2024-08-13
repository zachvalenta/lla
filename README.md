# `lla` - Advanced File Listing Utility

`lla` is a high-performance, extensible alternative to the traditional `ls` command, written in Rust. It offers enhanced functionality, customizable output, and a plugin system for extended capabilities.

![lla in action](https://github.com/triyanox/lla/blob/main/lla.png?raw=true)

## Features

- **Efficient file listing**: Optimized for speed, even in large directories
- **Multiple view modes**: 
  - Default view
  - Long format (`-l`)
  - Tree view (`-t`)
  - Recursive listing (`-R`)
- **Advanced sorting**: 
  - Alphabetical (default)
  - File size (`-s size`)
  - Modification date (`-s date`)
- **Flexible filtering**: Filter by filename or extension (`-f, --filter`)
- **Customizable recursion**: Set maximum depth for subdirectory traversal (`-d, --depth`)
- **Extensible plugin system**: Develop and integrate custom functionality

## Installation

### From crates.io

```bash
cargo install lla
```

### For NetBSD users

```bash
pkgin install lla
```
(we see you, netbsd. we appreciate you.)

## how to play

```
lla [OPTIONS] [DIRECTORY]
```

### Core Options

- `-l, --long`: Use long listing format
- `-R, --recursive`: List subdirectories recursively
- `-t, --tree`: Display files in a tree structure
- `-s, --sort <CRITERIA>`: Sort by "name", "size", or "date"
- `-f, --filter <PATTERN>`: Filter files by name or extension
- `-d, --depth <DEPTH>`: Set maximum recursion depth

### Plugin Management

- `--enable-plugin <NAME>`: Enable a specific plugin
- `--disable-plugin <NAME>`: Disable a specific plugin
- `--plugins-dir <PATH>`: Specify custom plugins directory

### Utility Commands

- `lla install`: Install plugins
  - `--git <URL>`: Install from a Git repository
  - `--dir <PATH>`: Install from a local directory
- `lla list-plugins`: Display all available plugins
- `lla init`: Initialize configuration file

## Configuration

`lla` uses a TOML configuration file located at `~/.config/lla/config.toml`. Initialize with default settings:

```bash
lla init
```

Example configuration:

```toml
default_sort = "name"
default_format = "default"
enabled_plugins = ["git_status", "file_hash"]
plugins_dir = "/home/user/.config/lla/plugins"
default_depth = 3
```

## Install Plugins

You can instll plugins from a local directory or from a Git repository.

You can find official plugins [here](
    https://github.com/triyanox/lla/blob/main/plugins.md
).


### From Git

```bash
lla install --git <github_url>
```

### From Local Directory

```bash
lla install --dir <path>
```

## Plugin Development

Develop custom plugins to extend `lla`'s functionality. Plugins are dynamic libraries that implement the `Plugin` trait from the `lla_plugin_interface` crate.

### Plugin Structure

1. Create a new Rust library:
   ```bash
   cargo new --lib my_lla_plugin
   ```

2. Add dependencies to `Cargo.toml`:
   ```toml
   [dependencies]
   lla_plugin_interface = { path = "/path/to/lla_plugin_interface" }
   
   [lib]
   crate-type = ["cdylib"]
   ```

3. Implement the `Plugin` trait:

```rust
use lla_plugin_interface::{Plugin, DecoratedEntry, EntryDecorator};

pub struct MyPlugin;

impl Plugin for MyPlugin {
    fn name(&self) -> &'static str {
        "my_plugin"
    }

    fn version(&self) -> &'static str {
        env!("CARGO_PKG_VERSION")
    }

    fn description(&self) -> &'static str {
        env!("CARGO_PKG_DESCRIPTION")
    }
}

impl EntryDecorator for MyPlugin {
    fn decorate(&self, entry: &mut DecoratedEntry) {
        // Add custom fields or modify entry
    }

    fn format_field(&self, entry: &DecoratedEntry, format: &str) -> Option<String> {
        // Return formatted string for display
    }

    fn supported_formats(&self) -> Vec<&'static str> {
        vec!["default", "long", "tree"]
    }
}

lla_plugin_interface::declare_plugin!(MyPlugin);
```

4. Build your plugin:
   ```bash
   cargo add lla_plugin_interface
   cargo build --release
   ```

5. Install the plugin:
   ```bash
   lla install --dir /path/to/my_lla_plugin
   ```
    or
    ```bash
    lla install --git <git_repo>
    ```


### Plugin Interface

The `lla_plugin_interface` crate provides the following key components:

- `Plugin` trait: Core interface for plugin functionality
- `EntryDecorator` trait: Methods for decorating and formatting file entries
- `DecoratedEntry` struct: Represents a file entry with metadata and custom fields

Refer to the `lla_plugin_interface` documentation for detailed API information.

## Examples

```bash
# Long format, sorted by size, showing only .rs files, max depth 3
lla -ls size -f .rs -d 3

# enable git status plugin
lla --enable-plugin git_status
# or for multiple plugins
lla --enable-plugin git_status categorizer

# disanble git status plugin
lla --disable-plugin git_status
# or for multiple plugins
lla --disable-plugin git_status categorizer
```

## Contributing

Contributions are welcome! Please feel free to submit pull requests, report bugs, and suggest features.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -m 'Add some new-feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/triyanox/lla/blob/main/LICENSE) file for details.