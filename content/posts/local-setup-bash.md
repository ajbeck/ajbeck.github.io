---
title: 'My Local Bash Setup'
date: 2025-03-01T20:00:00+01:00
draft: true
---

The nature of what I do has me doing a lot of things in `bash`. A bit ago, after causing a few hiccups 😄, I decided it was time to just use `bash` as my daily shell. Which meant I had the joy of setting it up on macOS. Roughly, here is how I accomplished it:

1. Install latest `bash` (macOS 15 ships with v3.2 🫣)
2. Add homebrew's `bash` to the shell allow list
3. Change my default shell
4. Setup `.bash_profile` and `.bashrc`

## Install Latest bash

```bash
brew install bash
```

macOS does not ship with the latest version of `bash`, as I'm writing this the version from macOS is v3.2. To install the latest I'm using homebrew.

## Using homebrew's bash

To use a shell as your default shell it must be 'allow listed'. The allow list is located in the file `/etc/shells`, you will need `sudo` permissions to modify this file.

```bash
echo "$(brew --prefix)/bin/bash" | sudo tee -a /etc/shells

cat /etc/shells
```

## Set my default shell

```bash
chsh -s $(brew --prefix)/bin/bash
```

## .bash_profile and .bashrc

There are two files to be aware of when working to load/manage bash's runtime configuration, `.bash_profile` and `.bashrc`. In macOS (darwin linux):

- `~/.bash_profile` is executed once when a login session is stared, generally when a login shell is opened
- `~/.bashrc` is executed once each time a new shell or sub-shell is opened

---

### References {class="text-gray-700"}

<ul class="list-none">
  <li class="text-sm">
    Bash StartUp Files</br>
    <a href="">https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html</a>
  </li>
</ul>
