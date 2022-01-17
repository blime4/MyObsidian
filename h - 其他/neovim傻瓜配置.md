














# Setup

## Pre-requisites[#](https://nvchad.github.io/getting-started/setup#pre-requisites "Direct link to heading")

-   [Neovim 0.6.1 install page](https://github.com/neovim/neovim/releases/tag/v0.6.1)

(note : use the stable 0.6v from the releases only i.e the same version if packaged in your distro will work too, dont use nightly! )

-   [Use a Nerd Font](https://www.nerdfonts.com/) in your terminal emulator.

## Install[#](https://nvchad.github.io/getting-started/setup#install "Direct link to heading")

Installation is as easy as cloning the NvChad repository into your NeoVim configuration folder.

Next, clone the `NvChad` repository to `~/.config/nvim` & install plugins with:

-   Linux & MacOS
-   Windows

### Linux & MacOS:[#](https://nvchad.github.io/getting-started/setup#linux--macos "Direct link to heading")

If you already have a `~/.config/nvim` folder, make a backup with:

```
mv ~/.config/nvim ~/.config/NVIM.BAK
```

Copy

Then install NvChad & it's plugins with:

```
git clone https://github.com/NvChad/NvChad ~/.config/nvim --depth 1
nvim +'hi NormalFloat guibg=#1e222a' +PackerSync
```

Copy

## Update[#](https://nvchad.github.io/getting-started/setup#update "Direct link to heading")

NvChad has an update mechanism built-in, which will pull any new updates to the git repository.

Activate it by running `<leader> + uu`.

-   Note: by NvChad default, `<leader>` is the `<space>` key

This will open a prompt in NeoVim warning you that it is about to do a `git reset --hard` and **you will lose any customisations** you've made to NvChad that are not in designated customisation folders.

## Uninstall[#](https://nvchad.github.io/getting-started/setup#uninstall "Direct link to heading")








## Uninstall[#](https://nvchad.github.io/getting-started/setup#uninstall "Direct link to heading")

Uninstalling is as simple as removing the `nvim` configuration directories.

> Note: it's suggested to backup your config first, consider `mv ~/.config/nvim ~/.config/NVIM.BAK`

```
rm -rf ~/.config/nvim
rm -rf ~/.local/share/nvim
rm -rf ~/.cache/nvim
```