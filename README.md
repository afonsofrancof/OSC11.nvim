# OSC11.nvim

A *tiny* Neovim plugin for automatic terminal theme synchronization via OSC 11.

When your terminal theme changes, Neovim detects it, asks the terminal for its new background color via OSC 11, and updates its internal `background` option based on whether that color is light or dark.

`OSC11.nvim` hooks into that same OSC 11 response and uses the same light/dark detection logic as Neovim.  
The difference is that instead of only updating `background`, it lets you run any Lua function when the terminal switches between light and dark modes.

> [!NOTE]
> This project is feature complete for its intended small scope of managing Git worktrees (create, delete, switch). While actively maintained (don't be scared if there are no commits for a long time), new features will be limited to improvements of existing functionality and simple additions within this scope. Pull requests are welcome as long as they align with these goals.

## Features

- Automatic detection of terminal light/dark theme changes via OSC 11.
- Configurable callbacks for light and dark theme switches.

## Requirements

- Neovim 0.11.0 or higher.
- A terminal that supports OSC 11 responses (e.g., Ghostty).
- If using tmux, you might need `allow-passthrough` set to `on` in `tmux.conf`.

```conf
set -g allow-passthrough on
```

## Installation

### Setup with [lazy.nvim](https://github.com/folke/lazy.nvim)

```lua
{
   "afonsofrancof/OSC11.nvim",
    opts = {
       -- Function to call when switching to dark theme
       on_dark = function()
        vim.opt.background = "dark"
        vim.cmd('colorscheme gruvbox-material')
       end,
       -- Function to call when switching to light theme
       on_light = function()
         vim.cmd("colorscheme tokyonight-day")
       end,
    }
}
```

### Configuration Options

| Option     | Description                                   | Default |
| :--------- | :-------------------------------------------- | :------ |
| `on_dark`  | Function to call when switching to dark theme | `nil`   |
| `on_light` | Function to call when switching to light theme | `nil`   |

## Usage

After setting up the plugin, `OSC11.nvim` will automatically listen for `TermResponse` events. When a terminal theme change is detected via an OSC 11 sequence, the corresponding `on_dark` or `on_light` callback will be executed.

### Example

To demonstrate, if you have a terminal that changes its theme based on the system's light/dark mode, and you've configured `OSC11.nvim` as shown in the installation section, your Neovim colorscheme will automatically switch between `gruvbox-material` and `tokyonight-day` when your terminal theme changes.

## How It Works

1.  **OSC 11 Listener**: The plugin sets up an `Autocmd` on the `TermResponse` event. This event is triggered when the terminal sends a response to a query, which includes OSC 11 responses.
2.  **Parse OSC 11 Response**: When a `TermResponse` is received, the plugin checks if it's an OSC 11 sequence. An OSC 11 sequence (e.g., `^[]11;rgb:RRRR/GGGG/BBBB^[\`) contains the current background color of the terminal.
3.  **Calculate Luminance**: The RGB values from the OSC 11 response are converted to decimal, and a luminance value is calculated. This luminance calculation is the same as Neovim's internal method for determining if a background is light or dark.
4.  **Determine Theme**: If the calculated luminance is less than 0.5, the theme is considered "dark"; otherwise, it's "light."
5.  **Execute Callbacks**: Based on the determined theme ("dark" or "light"), the corresponding `on_dark` or `on_light` function provided in the configuration is executed. Finally, `vim.cmd("redraw!")` is called to ensure the Neovim UI is updated.

## License

MIT
