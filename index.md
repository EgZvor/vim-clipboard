# Comprehensive guide to Vim's clipboard support

This guide assumes you know the *basics* of Vim.

## Environment support

If you're on **Linux** I assume you use X11.
That doesn't mean the guide will not work on Wayland.

For now this guide is for **Vim** only, not **Neovim**.
Even though the usage is the same, the inner workings are quite different.
The good news is that in Neovim there shouldn't be a need to set anything up
as far as I know, so you can skip to [The `+` register](#the-register).

In Vim native clipboard support is *optionally* included at compile time —
you may not have it.

Test if it already works: click (gasp!) on some word, press `"+yiw`
(yes, that's a double quote, then plus and then `yiw`)
and try to paste in another program.
If it worked skip to [The `+` register](#the-register).
If you're working over SSH,
learn to use clipboard support locally first,
then see [Copying/pasting over SSH](#copyingpasting-over-ssh).

## Check native clipboard support

To see if your Vim has clipboard support compiled-in type

```
:echo has('clipboard')
```

if it outputs `1`, clipboard support is enabled,
if `0` — then not.

A less convenient but easier to remember command is `:version`.
It outputs all the optional features along with an indicator:
`+` or `-` that they are enabled. Look for `+clipboard`.

If you're using **Windows** Subsystem for Linux also
[ensure it has X11 support](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps).

### How to enable clipboard support

On Linux distributions there are usually multiple Vim packages provided:
some minimal, some more feature-rich.

For example, on **Arch Linux** there are `vim` and `gvim` packages.
The second not only includes the GUI version of Vim,
but a more feature-rich TUI.
One that includes clipboard support.

On the off chance there is no such "big" version of Vim available
you can compile it yourself.
Look at the `src/INSTALL` file in the Vim repository.
Use this flag on the configuration step:

```shell
./configure --with-features=huge
```

## The `+` register

In Vim you can keep more than 26 pieces of text in memory at the same time.
They are kept in [registers](https://vimhelp.org/change.txt.html#registers).

For [operators](https://vimhelp.org/motion.txt.html#operator) such as `y`ank, `d`elete and `p`aste
you can optionally specify a register by prefixing it with a double quote
and a one-character name of the register.
For example, `"ryy` will yank the current line into the `r` register.
Afterwards you can paste it with `"rp`.

Apart from the 26 letter registers there are some special ones.

If you *don't* specify a register the *default* register is used.
You can also access it with the `"` name, for example, `""yiw`.
This will work exactly the same as `yiw` by default.
However the `"` register is still useful to paste in **Insert mode** with `<c-r>"`.
The default register is always populated,
even if you specify another register.

Registers related to clipboard are `*` and `+`.
On **Mac** and **Windows** those are the same.
On **Linux** `*` and `+` correspond to PRIMARY and CLIPBOARD selections.

For example, select some text with a mouse in a browser,
go to Vim and use `"*p` to paste it.

Or, use **Visual mode** to select some text in Vim, press `"+y`,
go to another program and paste it with `CTRL+V`.

Here are some ways to make working with system clipboard more smooth.

### unnamedplus

There is the ['clipboard'](https://vimhelp.org/options.txt.html#%27clipboard%27) option.
If you set it to `unnamed`,
whenever you *don't* specify a register the `*` register is going to be used.
If you set it to `unnamedplus`, the `+` register is used.
Note that the system clipboard is then overwritten whenever you `d`elete something.

```vim
set clipboard^=unnamedplus
```

### Mappings

Another common approach is to utilize mappings.
You can put this line into your .vimrc file and see if you like it.

```vim
nnoremap <space>p "+
```

Whenever you want to paste some text from another program press `<space>pp`.
Whenever you want to copy text from Vim you can do something like `<space>pyG`
to copy text from cursor position up to the end of the file.

## Copying/pasting over SSH

To make native clipboard support work over SSH on **Linux** you need to enable X11 forwarding.

Alternative solutions:

- [vim-oscyank](https://github.com/ojroques/vim-oscyank)
- `rsync` your files
- use [SSHFS](https://en.wikipedia.org/wiki/SSHFS)
- [terminal paste](#fallback-solution---terminal-paste)

## Fallback solution - terminal paste

You can treat Vim like any other terminal program without built-in clipboard support.

Use middle-click or CTRL+SHIFT+V for pasting into Vim
(or whatever your terminal's paste binding is).
Hold SHIFT and select text with a mouse for a
[modeless-selection](https://vimhelp.org/gui.txt.html#modeless-selection),
then press CTRL+SHIFT+C
(or whatever your terminal's copy binding is).

Another common way is using a "copy mode" of a screen multiplexer like `tmux`.

### Cons

- Paste of many lines is slow and may have artifacts.
- Selection is screen-wise,
  meaning you can't select text from one split window only.
- If your version of Vim is below 9-ish you have to
  `:set paste` before pasting and `:set nopaste` afterwards.
  See [vim-unimpaired](https://github.com/tpope/vim-unimpaired)'s ``yop`` for a solution.

### Pros

- Universal across terminal programs.
- Works over SSH, but see [Copying/pasting over SSH](#copyingpasting-over-ssh).

## Fallback solution - use CLI programs

Vim has multiple ways of interacting with CLI programs.

**Linux** examples (see
[How to Copy Text in Vim](https://ladedu.com/how-to-copy-text-in-vim-to-the-clipboard/)
for other systems):

To copy selected (in **Visual mode**) lines type this
(`'<,'>` will be entered automatically):

```vim
:'<,'>w !xclip -selection clipboard
```

To paste from the clipboard to a line below, type this:

```vim
:r !xclip -o -selection clipboard
```

- [:h write_c](https://vimhelp.org/editing.txt.html#%3Awrite_c)
- [:h read!](https://vimhelp.org/insert.txt.html#%3Aread%21)

### Cons

- Linewise changes only — no support for
  [text objects](https://vimhelp.org/motion.txt.html#text-objects).

### Pros

- Follows UNIX philosophy.
- CLI programs power **Neovim**'s
  [clipboard support](https://neovim.io/doc/user/provider.html#provider-clipboard),
  which means it is a robust solution for environments.

## Reference links

- [Repository](https://github.com/EgZvor/vim-clipboard)
- [Vim help](https://vimhelp.org/helphelp.txt.html#helphelp)
- [Vim Fandom Wiki](https://vim.fandom.com/wiki/Accessing_the_system_clipboard)
- More on registers in [Vim help](https://vimhelp.org/change.txt.html#registers).
- [Vi Stack Exchange](https://vi.stackexchange.com/questions/84/how-can-i-copy-text-to-the-system-clipboard-from-vim)
- [How to Copy Text in Vim](https://ladedu.com/how-to-copy-text-in-vim-to-the-clipboard/)
