---
title: Mint Troubleshooting
navTitle: Mint Troubleshooting
---

# Mint troubleshooting

Mint is a package manager like `brew`, `bundle`, `apt-get`
https://github.com/yonaskolb/Mint

The applications it manages are listed in the [Mintfile](../../Mintfile)

# Running Mint bootstrap

Will then use `brew` to install `mint`. It will then install the
compatible versions of our required (swift based) command line tools linking
them into `/usr/local/bin`. 

If you have mint already installed you can obtain verbose output by running it like so:

```bash
mint bootstrap --link --verbose
```

### Skip linking (advanced)
You can skip linking by passing `--no-link` you will
then need to run the tools by using the `mint run` prefix (since they are
unlikely to be in your $PATH)

## Common Errors

### Script Hangs
This script uses `git+ssh` to clone some internal tools. Ensure you've setup
your ssh keys for `gecgithub01.walmart.com`. You can validate your setup by
running `ssh -T git@gecgithub01.walmart.com`)

### Failed to resolve `glass-swift-scripts`

This isn't necessarily a mint issue, but it is prone to happen when running `./scripts/setup-mint.sh`, the issue originates from
Xcode build tools but it _looks_ as if was caused by `mint`, output is the following:

```bash
# ... other output ...
🌱 Resolving package
error: terminated(72): TMPDIR=/var/folders/ys/xyssbjk56mz2_qwm1nxhf50c0000gp/T/ PAGER=less LSCOLORS=Gxfxcxdxbxegedabagacad PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin LC_TERMINAL_VERSION=3.4.4 XPC_FLAGS=0x0 SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.Pc1fRwH4eg/Listeners USER=vn51sxj _=/usr/bin/swift COLORTERM=truecolor TERM_PROGRAM=iTerm.app CPATH=/usr/local/include __CF_USER_TEXT_ENCODING=0x1F6:0x0:0x0 TERM_PROGRAM_VERSION=3.4.4 LESS=-R XPC_SERVICE_NAME=0 ITERM_PROFILE=Default ITERM_SESSION_ID=w0t1p0:A2762C3A-8303-48FC-9E92-DB9C660A57DB PWD=/private/var/folders/ys/xyssbjk56mz2_qwm1nxhf50c0000gp/T/mint/git_gecgithub01.walmart.com_walmart-ios_glass-swift-scripts SHELL=/bin/zsh LC_TERMINAL=iTerm2 COLORFGBG=7;0 SDKROOT=/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk HOME=/Users/vn51sxj SHLVL=2 LOGNAME=vn51sxj ZSH=/Users/vn51sxj/.oh-my-zsh TERM=xterm-256color COMMAND_MODE=unix2003 LIBRARY_PATH=/usr/local/lib TERM_SESSION_ID=w0t1p0:A2762C3A-8303-48FC-9E92-DB9C660A57DB LANG=en_US.UTF-8 /usr/bin/xcrun --sdk macosx --find xctest output:
    xcrun: error: unable to find utility "xctest", not a developer tool or in PATH

🌱 Encountered error during "swift package resolve"
🌱  Failed to resolve glass-swift-scripts v0.8.1 with SPM
```

To fix it make sure to have installed the recommended Xcode version from the main readme and in the terminal that is
running mint set the Xcode location:

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```
### Unable to access https://github.com
You might be connecting to a VPN that does not allow "external traffic".
Try connecting to **WeC Two Factor VPN**

### Brew Missing
Brew is a pretty popular package manager for macOS.
https://brew.sh

You can find [install directions here](https://brew.sh).

