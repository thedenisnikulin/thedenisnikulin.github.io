+++
title = 'My Approach to Neovim'
date = 2023-07-06T00:00:00+00:00
draft = false
+++

# The story

I always found myself looking towards those terminal based modal editors like Neovim, but every time one thing really held me behind: the amount of time I needed to put into configuration. You have to spend quite a lot of time to configure a stable feature-complete version of Neovim, and it actually doesn't quite pay off in the beginning, because it's so tempting to stay with your comfy VS Code and don't waste time. I tried to configure Neovim like 3 times in my entire career, every time with a goal to build a complete editor, but having successfully accomplished it only with my latest attempt.

When I was most disappointed configuring Neovim, I have found Helix, a Neovim alternative with lots of features built in. It really made me interested when I first heard about it. Because configuration is minimal, I could focus more on the editing itself without thinking much about what plugin I need to install next, and so Helix became the editor which introduced modal editing to me. When using Neovim in the beginning, I was more focused on just making it look good ("tUrn yOuR vIm iNto a fUlL fEaTuRed IdE" kind of videos, you know), but I didn't study much the "vim way" of editing. I was using only some keys like `hjkl`, maybe some `w` and `b`, sometimes mouse, which by themselves are very far from making the editing experience productive. Though I eventually left Helix, it contributed much to how I approached modal editing.

After becoming comfortable with Helix, some crazy idea popped up in my head: "now if I am good at Helix, and Helix is like Neovim, what if I try to configure Neovim similarly to Helix? Then, with powerful plugin system, I could gradually improve my experience, the thing I cannot do with Helix which lacks plugins". After thinking a little, I've cleaned my old `~/.config/nvim` directory, and got my hands dirty. And to my very surprise, the idea worked like a charm: I actually went to a minimally viable working Neovim config with essential plugins and features in a single `init.lua` file really, really fast! It took me nearly a couple of days to configure without much pain, unlike my any previous attempt. Great!

I think why it really worked for me this time is because my requirements for an editor were much more "real" when I switched from Helix to Neovim. When you switch from something like VSCode, you want those features that GUI editors have, which are mostly very different from what you get in modal editors. You switch the editing paradigm which you haven't yet understood and accepted, and that's why you expect something from Neovim which it cannot provide (for good though). That's why some people misinterpret Neovim as being an IDE, because they want to see it as an IDE, which eventually lead to frustration.

What I learned from this is that Neovim is a different code editor. And if you want to learn it well, you should approach it differently, as a new thing, without bringing your old habits from traditional editors.

# The approach
Below are the points that changed Neovim for me. Having all these makes me really happy with my current configuration.

## 1. Keymaps
I really like the way keymaps are organized in Helix. So my keymaps are organized in a similar way.
Basically, I have 2 types (in Helix they are called "modes") of keymaps - `goto` and `space` keymaps. Goto are those keymaps which make you jump somewhere, like "Go to definition", "Go to next buffer", etc. Space are general purpose (space because it's the leader key).

My space keymaps (not exhaustive):
![My space keymaps (not exhaustive)](/8eqmtlv5iecb07emovmv.png)

My goto keymaps:
![My goto keymaps](/vx1pgnrs8xvc10f13rlz.png)

## 2. Much of Telescope
You can use Telescope for so much of things! File browser, notification history, keymaps browser, filetypes picker, diagnostics, git, DAP command picker, and so on. It's a such fundamental tool that I can't imagine using Neoivm without it.

It just fits so good with Neovim. I don't really like split windows because they are somewhat clumsy in terminal editors, that's why I don't have `nerdtree` and don't use `nvim-dap-ui`: I find such "util splits" not so pleasant to work with. Telescope file browser and `dap.ui.widgets` are just much better (though the latter is not Telescope, but neither is a split window). And a lot of things can be implemented with Telescope. It contributes a very huge part to overall editing experience.

Telescope File browser
![Image description](/xg5jl0r3cxwoeoe73y2f.png)

Git commits with Telescope
![Image description](/10p1n4ktcj36uqbtgs2h.png)

DAP commands with Telescope
![Image description](/ufkh7oehwfkuhvxgh8hx.png)


## 3. Enhanced movement
There's not much to say: using `leap.nvim` adds up to editing speed on mid-range movements, can't really edit in nvim without it.

## 4. Nice UI
Not related to "the approach", but made me enjoy the editor much more

Normal mode commands, search, code actions at screen center with nice round corners
![Image description](/z1mhxvrq68cp9oww7zb8.png)

![Image description](/0c5k543n38sxnav0axhf.png)

![Image description](/6qnleduueeo9jcr1ay65.png)

Pretty notifications

![Image description](/ar49akbux5mg31a0nbeo.png)

## 5. Use other tools
I think Neovim is a very powerful editor, but it's still and editor, and sometimes you need more than this. You have so much CLI tools, and the power of pipes, use it to complement your Neovim experience. The following are just some from the top of my head:
- `jq` for json manipulations (using nvim pipes with jq is really cool, for example, you can pipe visually selected json string to `jq` which will effectively format it)
- `tmux` or `zellij` for workspace management
- [`ast-grep`](https://github.com/ast-grep/ast-grep) - helps so much in refactoring.
- and pretty much anything you can use in your terminal.

## 6. No abstraction in config
While it would be quickly tempting for perfectionist minds to start writing custom functions for mapping keys to use everywhere ("wait, `nvim-cmp` has its own keymaps table which differs from standard `vim.keymap.set` function? Hell no, I must write a function that abstracts it away!"), I saved so much time avoiding perfectionism and abstractions by writing configuration in a "dumb" way. If you look at my config, you will notice that it's not quite smart, there are no custom functions, some settings I just copy-pasted from somebody else's config. I think you should not waste time on this, unless you are writing a Neovim distro (God be with you: never found them actually any useful).

## 7. Start with starting configs if you are lost
Initially, I just copy pasted [this config](github.com/nvim-lua/kickstart.nvim) and deleted everything I didn't need, gradually adding things adding up. I don't really like Neovim distributions, though you can use them on the early stages of your journey, but make sure to jump off that ship when you feel more confident. 

# Conclusion
I've learned that you get most power from Vim if you approach it the right way. The editor makes you customize your experience as you want, and it really feels different when you get it. Understanding Neovim got me starting my journey of customizing the perfect dev workflow, and I help this article will help you too. :)

Link to my config: https://github.com/thedenisnikulin/nvim
