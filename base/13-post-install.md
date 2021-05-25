# Post-installation setup

## Sanity check configurations

Ensure you can `ssh` to the host as `lfs` by grabbing the ip address from the `Hyper-V` manager and attempting. Once in, do a few quick checks, such as pinging a popular web url to ensure your network connection works, checking the `/etc/os-release` file to ensure you're actually in the `lfs` system, and checking the status of various services and the kernel with `systemctl status`, `journalctl -xe`, and `dsmseg`. Don't expect no error messages, but all expected services should at least be working.

## Generate the trust store

Note that we installed the `make-ca` package earlier, in order to install trusted CAs designated by the Mozilla Foundation, but because the system had no network connection, the CAs could not be installed. Now they can be, so run:

```sh
sudo make-ca -g
```

## Rebuid cURL

At this point, we have a CA trust store generated, so `cURL` can be rebuilt to use it:

```sh
tar xzvf curl-7.76.1.tar.gz
cd curl-7.76.1

./configure --prefix=/usr    \
            --with-openssl   \
            --with-gnutls    \
            --disable-static \
            --with-ca-bundle=/etc/pki/tls/certs/ca-bundle.crt
make -j
make test
sudo make install
sudo rm -v /usr/lib/libcurl.{,l}a

cd ..
rm -rf curl-7.76.1
```

## Cleanup permissions

There are a few leftover vestiges of using the `lfs` user from the original host system, i.e. the `/` directory itself, plus everything under `/sources`, is owned by some UID > 1000 that will not exist in the new system, so `chown` these to be owned by `root` instead:

```sh
sudo chown root:root /
sudo chown -R root:root /sources
```

## vim configuration

Create a minimal `~/.vimrc`. You can customize this if you have experience. See [Example vimrc](https://vim.fandom.com/wiki/Example_vimrc) from vim fandom wiki.

```sh
cat > ~/.vimrc << EOF
" URL: http://vim.wikia.com/wiki/Example_vimrc
" Authors: http://vim.wikia.com/wiki/Vim_on_Freenode
" Description: A minimal, but feature rich, example .vimrc. If you are a
"              newbie, basing your first .vimrc on this file is a good choice.
"              If you're a more advanced user, building your own .vimrc based
"              on this file is still a good idea.

"------------------------------------------------------------
" Features {{{1
"
" These options and commands enable some very useful features in Vim, that
" no user should have to live without.

" Set 'nocompatible' to ward off unexpected things that your distro might
" have made, as well as sanely reset options when re-sourcing .vimrc
set nocompatible

" Attempt to determine the type of a file based on its name and possibly its
" contents. Use this to allow intelligent auto-indenting for each filetype,
" and for plugins that are filetype specific.
if has('filetype')
  filetype indent plugin on
endif

" Enable syntax highlighting
if has('syntax')
  syntax on
endif

"------------------------------------------------------------
" Must have options {{{1
"
" These are highly recommended options.

" Vim with default settings does not allow easy switching between multiple files
" in the same editor window. Users can use multiple split windows or multiple
" tab pages to edit multiple files, but it is still best to enable an option to
" allow easier switching between files.
"
" One such option is the 'hidden' option, which allows you to re-use the same
" window and switch from an unsaved buffer without saving it first. Also allows
" you to keep an undo history for multiple files when re-using the same window
" in this way. Note that using persistent undo also lets you undo in multiple
" files even in the same window, but is less efficient and is actually designed
" for keeping undo history after closing Vim entirely. Vim will complain if you
" try to quit without saving, and swap files will keep you safe if your computer
" crashes.
set hidden

" Note that not everyone likes working this way (with the hidden option).
" Alternatives include using tabs or split windows instead of re-using the same
" window as mentioned above, and/or either of the following options:
" set confirm
" set autowriteall

" Better command-line completion
set wildmenu

" Show partial commands in the last line of the screen
set showcmd

" Highlight searches (use <C-L> to temporarily turn off highlighting; see the
" mapping of <C-L> below)
set hlsearch

" Modelines have historically been a source of security vulnerabilities. As
" such, it may be a good idea to disable them and use the securemodelines
" script, <http://www.vim.org/scripts/script.php?script_id=1876>.
" set nomodeline


"------------------------------------------------------------
" Usability options {{{1
"
" These are options that users frequently set in their .vimrc. Some of them
" change Vim's behaviour in ways which deviate from the true Vi way, but
" which are considered to add usability. Which, if any, of these options to
" use is very much a personal preference, but they are harmless.

" Use case insensitive search, except when using capital letters
set ignorecase
set smartcase

" Allow backspacing over autoindent, line breaks and start of insert action
set backspace=indent,eol,start

" When opening a new line and no filetype-specific indenting is enabled, keep
" the same indent as the line you're currently on. Useful for READMEs, etc.
set autoindent

" Stop certain movements from always going to the first character of a line.
" While this behaviour deviates from that of Vi, it does what most users
" coming from other editors would expect.
set nostartofline

" Display the cursor position on the last line of the screen or in the status
" line of a window
set ruler

" Always display the status line, even if only one window is displayed
set laststatus=2

" Instead of failing a command because of unsaved changes, instead raise a
" dialogue asking if you wish to save changed files.
set confirm

" Use visual bell instead of beeping when doing something wrong
set visualbell

" And reset the terminal code for the visual bell. If visualbell is set, and
" this line is also included, vim will neither flash nor beep. If visualbell
" is unset, this does nothing.
set t_vb=

" Enable use of the mouse for all modes
if has('mouse')
  set mouse=a
endif

" Set the command window height to 2 lines, to avoid many cases of having to
" "press <Enter> to continue"
set cmdheight=2

" Display line numbers on the left
set number

" Quickly time out on keycodes, but never time out on mappings
set notimeout ttimeout ttimeoutlen=200

" Use <F11> to toggle between 'paste' and 'nopaste'
set pastetoggle=<F11>


"------------------------------------------------------------
" Indentation options {{{1
"
" Indentation settings according to personal preference.

" Indentation settings for using 4 spaces instead of tabs.
" Do not change 'tabstop' from its default value of 8 with this setup.
set shiftwidth=4
set softtabstop=4
set expandtab

" Indentation settings for using hard tabs for indent. Display tabs as
" four characters wide.
"set shiftwidth=4
"set tabstop=4


"------------------------------------------------------------
" Mappings {{{1
"
" Useful mappings

" Map Y to act like D and C, i.e. to yank until EOL, rather than act as yy,
" which is the default
map Y y$

" Map <C-L> (redraw screen) to also turn off search highlighting until the
" next search
nnoremap <C-L> :nohl<CR><C-L>

"------------------------------------------------------------
EOF
```

Ensure the `EDITOR` environment variable is set:

```sh
echo "EDITOR=vim" >> ~/.zshrc
```

## zsh syntax highlighting

Grab the latest tagged release from `GitHub`. Run, from `/sources`:

```sh
curl -L https://github.com/zsh-users/zsh-syntax-highlighting/archive/refs/tags/0.7.1.tar.gz -o zsh-syntax-highlighting-0.7.1.tar.gz
tar xzvf zsh-syntax-highlighting-0.7.1.tar.gz
cd zsh-syntax-highlighting-0.7.1

sudo make PREFIX=/usr install

cd ..
rm -rf zsh-syntax-highlighting-0.7.1
```

Now, add this to your `~/.zshrc` (THIS MUST BE THE LAST LINE!):

```sh
echo ". /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ~/.zshrc
```

Optionally, you can add some extra completion functions that haven't made it to the main `zsh` package yet:

```sh
curl -L https://github.com/zsh-users/zsh-completions/archive/refs/tags/0.33.0.tar.gz -o zsh-completions-0.33.0.tar.gz
tar xzvf zsh-completions-0.33.0.tar.gz
cd zsh-completions-0.33.0

sudo install -vDm644 src/* /use/share/zsh/
```

## Setup cURL to follow redirects by default

Many web links to software packages, especially `GitHub` releases, utilize redirects. Rather than trying to remember which do or fix a broken download after the fact, we will just tell `cURL` to always follow redirects:

```sh
echo "-L" >> ~/.curlrc
```