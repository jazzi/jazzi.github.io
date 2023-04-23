---
layout: post
---

It's as easy as pie to get Homebrew installed on MacOS if follow the instructions on the official website, just one command:

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

However you need to install the Developer Command Line Tools in advance by excuting the following command:

`xcode-select --install`

## GFW problem

If you do as above everything should be fine but there is an exception, in China you will face similar errors as below:

> curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused

In short this url is blocked, the way to solve it is using resource within the wall, so excute the following script instead:

`/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"`

Then pick the mirror and the installation should be quite successful.

The way to remove Homebrew is quite simple too:

`/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"`

## No remote origin

When you type `brew udpate` or `brew doctor`, the following error will show up:

> Warning: No remote 'origin' in /opt/homebrew/Library/Taps/homebrew/homebrew-core, skipping update!

The way to solve it is as below:

`rm -rf "/opt/homebrew/Library/Taps/homebrew/homebrew-core"`

`brew tap homebrew/core`

And that's it!
