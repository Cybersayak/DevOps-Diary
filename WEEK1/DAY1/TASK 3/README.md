# Text Editor Mastery: Complete Vim Mastery Guide

## Table of Contents
1. [Foundation Concepts](#foundation-concepts)
2. [Vimtutor Mastery Path](#vimtutor-mastery-path)
3. [Vim Configuration](#vim-configuration)
4. [Multiple File Editing](#multiple-file-editing)
5. [System Config File Editing](#system-config-file-editing)
6. [Muscle Memory Drills](#muscle-memory-drills)
7. [Real-World DevOps Scenarios](#real-world-devops-scenarios)
8. [Advanced Techniques](#advanced-techniques)

---

## Foundation Concepts

### Why Vim Matters in DevOps

```
┌─────────────────────────────────────────────────────────┐
│  Why DevOps Engineers Must Master Vim:                  │
├─────────────────────────────────────────────────────────┤
│  ✓ Available on EVERY Linux/Unix server (even minimal) │
│  ✓ Edit files over SSH without GUI tools               │
│  ✓ Extremely fast for repetitive edits                 │
│  ✓ Powerful search/replace across multiple files       │
│  ✓ Integrates with shell commands                      │
│  ✓ Essential for emergency server troubleshooting      │
│  ✓ Edit config files: nginx, apache, systemd, etc.     │
│  ✓ Industry standard for infrastructure work           │
└─────────────────────────────────────────────────────────┘
```

### The Vim Philosophy

```
┌──────────────────────────────────────────────────────────┐
│  Vim Is Different From Normal Editors:                   │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  MODAL EDITING (The Core Concept):                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │  NORMAL MODE  → Navigate, delete, copy, paste   │    │
│  │     ↓ i,a,o                                      │    │
│  │  INSERT MODE  → Type text (like normal editors) │    │
│  │     ↓ Esc                                        │    │
│  │  NORMAL MODE  → Back to navigation               │    │
│  │     ↓ : (colon)                                  │    │
│  │  COMMAND MODE → Save, quit, advanced commands   │    │
│  │     ↓ v,V                                        │    │
│  │  VISUAL MODE  → Select text visually            │    │
│  └─────────────────────────────────────────────────┘    │
│                                                           │
│  Why Modal?                                              │
│  - Keyboard efficiency (no Ctrl+key combos needed)      │
│  - Each mode optimized for specific task                │
│  - Faster than mouse once mastered                      │
└──────────────────────────────────────────────────────────┘
```

### Visual Mode Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    VIM MODE FLOW                         │
└─────────────────────────────────────────────────────────┘

    ┌──────────────┐
    │ NORMAL MODE  │  ← Default mode (navigation, commands)
    │ (Gray cursor)│
    └──────┬───────┘
           │
     ┌─────┼─────┬──────┬──────┐
     │     │     │      │      │
     i     a     o      v      :
     │     │     │      │      │
     ↓     ↓     ↓      ↓      ↓
   ┌──┐  ┌──┐  ┌──┐  ┌───┐  ┌────┐
   │I │  │I │  │I │  │VIS│  │CMD │
   │N │  │N │  │N │  │UAL│  │    │
   │S │  │S │  │S │  │   │  │    │
   └──┘  └──┘  └──┘  └───┘  └────┘
    ↓     ↓     ↓      ↓      ↓
   Esc   Esc   Esc    Esc   Enter
    │     │     │      │      │
    └─────┴─────┴──────┴──────┘
              ↓
    ┌──────────────┐
    │ NORMAL MODE  │
    └──────────────┘
```

---

## Vimtutor Mastery Path

### Part 1: Understanding Vimtutor

**What is Vimtutor?**
```bash
# Launch vimtutor
vimtutor

# What happens:
# 1. Copies tutorial file to /tmp
# 2. Opens in vim with instructions
# 3. Interactive lessons you edit directly
# 4. Takes ~30 minutes per run

# Exit anytime with: :q
```

**Why 3 Times?**
```
┌─────────────────────────────────────────────────────┐
│  LEARNING PROGRESSION:                              │
├─────────────────────────────────────────────────────┤
│  Run 1 (30 min): Learn concepts                     │
│    - Confused, lots of mistakes                     │
│    - Reading instructions carefully                 │
│    - Building initial understanding                 │
│                                                      │
│  Run 2 (20 min): Build familiarity                  │
│    - Recognize patterns                             │
│    - Less reading, more doing                       │
│    - Starting to feel natural                       │
│                                                      │
│  Run 3 (15 min): Muscle memory                      │
│    - Commands feel automatic                        │
│    - Focus on speed, not understanding              │
│    - Ready for real editing                         │
└─────────────────────────────────────────────────────┘
```

### Part 2: Vimtutor Breakdown - Lesson by Lesson

#### Lesson 1: Basic Movement

```
┌─────────────────────────────────────────────────────┐
│  CORE MOVEMENT KEYS (Home row - ergonomic!)         │
├─────────────────────────────────────────────────────┤
│         k (up)                                       │
│          ↑                                           │
│  h (left) ← → l (right)                             │
│          ↓                                           │
│       j (down)                                       │
│                                                      │
│  WHY these keys?                                     │
│  - Your fingers rest on home row                    │
│  - No need to move hand to arrow keys               │
│  - Faster than mouse                                │
│                                                      │
│  MUSCLE MEMORY DRILL:                               │
│  hjkl hjkl hjkl (repeat 100 times)                  │
└─────────────────────────────────────────────────────┘
```

**Detailed Command Breakdown:**

```bash
# Open vimtutor
vimtutor

# Lesson 1.1: Moving the cursor
h   # Move LEFT one character
j   # Move DOWN one line
k   # Move UP one line
l   # Move RIGHT one character

# Practice pattern:
# 1. Place cursor at start of line
# 2. Press l l l l l (move right)
# 3. Press k k k (move up)
# 4. Press h h h (move left)
# 5. Press j j j (move down)

# Expected behavior:
# Cursor moves immediately (no Enter needed)
# Stays in NORMAL mode
```

**Why This Matters:**
```
Before Vim:
  Move cursor → Reach for mouse → Click → 1 second
  OR
  Move cursor → Reach for arrows → Press → 0.5 seconds

With Vim (hjkl):
  Move cursor → Press key → 0.1 seconds
  10x faster!

In DevOps:
  Editing /etc/nginx/nginx.conf (200 lines)
  Navigate to line 150 → Without Vim: 10 seconds
                      → With Vim: 150G (instant)
```

#### Lesson 1.2: Exiting Vim

```bash
# The famous "How to exit Vim" problem!

# Method 1: Quit without saving
:q
# Breakdown:
# : (colon)  → Enter COMMAND mode
# q          → Quit command
# Enter      → Execute

# Method 2: Quit and save
:wq
# w = write (save)
# q = quit

# Method 3: Force quit (discard changes)
:q!
# ! = force (ignore unsaved changes)

# Method 4: Save and quit (alternative)
ZZ
# (capital Z twice, in NORMAL mode)
# Faster than :wq
```

**Exit Vim Flowchart:**
```
Are you in INSERT mode?
        │
        ├─ YES → Press Esc → Go to next question
        │
        └─ NO  → Continue
                    │
        Have you made changes?
                    │
                    ├─ YES → Want to save?
                    │           │
                    │           ├─ YES → :wq
                    │           └─ NO  → :q!
                    │
                    └─ NO  → :q
```

#### Lesson 1.3: Deletion

```bash
# In NORMAL mode:

x   # Delete character UNDER cursor
    # Like pressing Delete key

# Example:
# Before: Hello World
#         ^(cursor on 'H')
# Press: x
# After:  ello World

# Practice:
# 1. Navigate to middle of word with h,j,k,l
# 2. Press x to delete character
# 3. Press u to undo (covered in Lesson 2)

# Delete multiple characters:
5x  # Delete 5 characters (current + next 4)

# Why x instead of Delete key?
# - Hands stay on home row
# - Can repeat with number prefix (5x)
# - Faster for multiple deletions
```

#### Lesson 1.4: Insertion

```bash
# Entering INSERT mode:

i   # Insert BEFORE cursor
    # (i = insert)

# Example:
# Text: Hello World
#       ^(cursor on 'H')
# Press: i
# Mode changes to: -- INSERT --
# Type: "Oh "
# Press: Esc (return to NORMAL mode)
# Result: Oh Hello World

a   # Append AFTER cursor
    # (a = append)

# Example:
# Text: Hello
#           ^(cursor on 'o')
# Press: a
# Type: " World"
# Press: Esc
# Result: Hello World

# Difference between i and a:
# i → Insert at cursor position (shifts existing text right)
# a → Insert after cursor (cursor moves right, then insert)
```

**Visual Comparison:**

```
TEXT: "Hello"
      01234 (positions)

Cursor on position 2 (l):

Press i:
  Hello
    ^(cursor)
  Type "XX"
  HeXXllo
    ^(cursor stays before original position)

Press a:
  Hello
    ^(cursor)
  Type "XX"
  HelXXlo
      ^(cursor moves after original position)
```

#### Lesson 2: Deletion and Undo

```bash
# Delete commands (NORMAL mode):

dw  # Delete WORD (from cursor to end of word)
    # d = delete operator
    # w = word motion

# Example:
# Text: Hello World Goodbye
#       ^(cursor)
# Press: dw
# Result: World Goodbye

d$  # Delete to END of line
    # $ = end of line motion

dd  # Delete ENTIRE line
    # Deletes line and moves lines below up

2dd # Delete 2 lines
    # Number prefix repeats command

# The Vim Grammar:
# [COUNT] OPERATOR MOTION
#   2        d        w     → Delete 2 words
#   3        d        d     → Delete 3 lines
#   1        d        $     → Delete to end of line
```

**Undo and Redo:**

```bash
u      # Undo last change
       # Can press multiple times to undo multiple changes

Ctrl+r # Redo (reverse undo)
       # Brings back undone changes

# Vim has UNLIMITED undo!
# Unlike many editors, you can undo back to file opening
```

#### Lesson 3: The Power of Operators and Motions

```bash
# Vim's Composable Commands:
# OPERATOR + MOTION = ACTION

# Operators:
d   # delete
c   # change (delete and enter INSERT mode)
y   # yank (copy)
p   # put (paste)

# Motions:
w   # word forward
b   # word backward
$   # end of line
0   # start of line
G   # end of file
gg  # start of file

# Combining:
dw  # delete word
d2w # delete 2 words
d$  # delete to end of line
d0  # delete to start of line
dG  # delete to end of file
dgg # delete to start of file

# Change command:
cw  # change word (delete word and enter INSERT mode)

# Example:
# Text: Hello World
#       ^(cursor on 'H')
# Press: cw
# Text: World (in INSERT mode)
# Type: "Goodbye"
# Press: Esc
# Result: Goodbye World
```

**Visual Motion Examples:**

```
TEXT: "The quick brown fox jumps over the lazy dog"
      ^(cursor on 'T')

dw → "quick brown fox jumps over the lazy dog"
     (deleted "The ")

d2w → "brown fox jumps over the lazy dog"
      (deleted "The quick ")

d$ → ""
     (deleted entire line from cursor)

dd → (empty line)
     (deleted entire line)
```

#### Lesson 4: Search and Replace

```bash
# Search:
/pattern    # Search FORWARD for pattern
            # Press Enter to jump to first match
            # Press n to go to NEXT match
            # Press N to go to PREVIOUS match

?pattern    # Search BACKWARD for pattern

# Example:
# In file with 100 lines, search for "server":
/server     # Press Enter
n           # Jump to next "server"
n           # Next again
N           # Go back to previous

# Replace:
:s/old/new/       # Replace first occurrence on current line
:s/old/new/g      # Replace all occurrences on current line (g = global)
:%s/old/new/g     # Replace all occurrences in entire file
:%s/old/new/gc    # Replace all, but ask for confirmation each time

# Detailed breakdown:
# :           → Enter COMMAND mode
# %           → All lines (range indicator)
# s           → Substitute command
# /old/new/   → Pattern and replacement
# g           → Global (all occurrences on line)
# c           → Confirm each replacement
```

**Real-World Example:**

```bash
# You're editing /etc/nginx/nginx.conf
# Need to change all instances of port 8080 to 8090

# Before:
# listen 8080;
# proxy_pass http://localhost:8080;
# upstream backend { server 127.0.0.1:8080; }

# Command:
:%s/8080/8090/g

# After:
# listen 8090;
# proxy_pass http://localhost:8090;
# upstream backend { server 127.0.0.1:8090; }

# Confirmation version (safer):
:%s/8080/8090/gc

# For each match, vim asks:
# replace with 8090 (y/n/a/q/l/^E/^Y)?
# y = yes, n = no, a = all, q = quit, l = this one and quit
```

#### Lesson 5: External Commands and File Operations

```bash
# Execute shell commands from vim:
:!command

# Example:
:!ls           # List files in current directory
:!pwd          # Print working directory
:!date         # Show current date/time

# Read output of command into file:
:r !command

# Example:
:r !date       # Insert current date at cursor position
:r !ls         # Insert directory listing

# Why this matters in DevOps:
# Editing script, need to check if command exists:
:!which nginx  # Without leaving vim!

# Insert system info into documentation:
:r !uname -a   # Insert system information
```

#### Lesson 6 & 7: Advanced Motions

```bash
# More precise movements:

e   # End of current word
E   # End of current WORD (ignores punctuation)

b   # Beginning of current/previous word
B   # Beginning of current/previous WORD

0   # Start of line (column 0)
^   # First non-whitespace character of line
$   # End of line

gg  # Go to first line of file
G   # Go to last line of file
50G # Go to line 50

%   # Jump to matching bracket: (), {}, []
    # Essential for code editing!

# Combining with operators:
d2e # Delete to end of next word
c$  # Change to end of line
y0  # Yank (copy) from start of line to cursor
```

### Part 3: Vimtutor Session Plan

#### Session 1 (30 minutes)

```bash
# Start fresh
vimtutor

# Focus areas:
# - Move with hjkl without thinking (Lesson 1)
# - Enter/exit INSERT mode confidently (Lesson 1)
# - Delete with x, dw, dd (Lesson 2)
# - Undo with u (Lesson 2)

# Success criteria:
# ✓ Can navigate without arrow keys
# ✓ Can enter text without confusion
# ✓ Can delete mistakes quickly
# ✓ Know how to exit vim (:q, :wq, :q!)
```

#### Session 2 (20 minutes)

```bash
# Launch vimtutor again
vimtutor

# Focus areas:
# - Operator + Motion combinations (Lesson 3)
# - Search with / and n (Lesson 4)
# - Simple search/replace (Lesson 4)
# - Recognize patterns from Session 1

# Success criteria:
# ✓ Commands feel familiar
# ✓ Can use d2w, c$, etc. without thinking
# ✓ Can search through file efficiently
# ✓ Complete in under 25 minutes
```

#### Session 3 (15 minutes)

```bash
# Final vimtutor run
vimtutor

# Focus areas:
# - Speed through all lessons
# - Don't read instructions, just execute
# - Focus on muscle memory, not understanding
# - Advanced motions (gg, G, %, etc.)

# Success criteria:
# ✓ Complete all lessons in under 20 minutes
# ✓ Commands are automatic
# ✓ Ready to edit real files
# ✓ Comfortable with vim's "language"
```

---

## Vim Configuration

### Part 1: Understanding .vimrc

**What is .vimrc?**
```
┌────────────────────────────────────────────────────┐
│  .vimrc = Vim Runtime Configuration                │
├────────────────────────────────────────────────────┤
│  Location: ~/.vimrc (in your home directory)       │
│                                                     │
│  Purpose:                                          │
│  - Set default options (line numbers, etc.)        │
│  - Define custom key mappings                      │
│  - Load plugins                                    │
│  - Set color schemes                               │
│                                                     │
│  Loaded automatically when vim starts              │
└────────────────────────────────────────────────────┘
```

**Creating Your First .vimrc:**

```bash
# Check if .vimrc exists
ls -la ~/.vimrc

# If not, create it
touch ~/.vimrc

# Open in vim
vim ~/.vimrc
```

### Part 2: Essential Configuration (Beginner)

**Basic .vimrc Configuration:**

```vim
" ~/.vimrc - Basic configuration
" Lines starting with " are comments

" ============================================
" DISPLAY SETTINGS
" ============================================

" Show line numbers
set number
" Why: Essential for navigating large files
"      Jump to specific line with :50 (go to line 50)

" Show relative line numbers (advanced)
set relativenumber
" Why: See distances to lines above/below
"      Example: Line 5 above shows as "5" instead of line number
"      Useful for: 5dd (delete 5 lines), 10j (move down 10)

" Highlight current line
set cursorline
" Why: Easy to see where you are in file
"      Visual indicator, especially in large files

" Show matching brackets
set showmatch
" Why: When typing ), vim briefly highlights matching (
"      Essential for code/config editing

" Enable syntax highlighting
syntax on
" Why: Color-codes different parts of code/config
"      Makes files easier to read

" ============================================
" INDENTATION SETTINGS
" ============================================

" Auto-indent new lines
set autoindent
" Why: New line starts at same indentation as previous
"      Manual indentation tedious

" Smart indentation
set smartindent
" Why: Vim recognizes code structure
"      Auto-indents after {, adjusts for }, etc.

" Convert tabs to spaces
set expandtab
" Why: Spaces more consistent across editors
"      Tabs display differently in different programs

" Set tab width to 4 spaces
set tabstop=4
" Why: 4 spaces is common standard
"      Readable but not too wide

" Set indentation width
set shiftwidth=4
" Why: Used by >> and << commands
"      Matches tabstop for consistency

" ============================================
" SEARCH SETTINGS
" ============================================

" Highlight search results
set hlsearch
" Why: All matches highlighted after search
"      Easy to see all occurrences

" Incremental search (search as you type)
set incsearch
" Why: Jumps to match while typing search
"      Faster than waiting for Enter

" Ignore case in searches
set ignorecase
" Why: /hello finds "Hello", "HELLO", "hello"
"      More forgiving

" Smart case (override ignorecase if search has uppercase)
set smartcase
" Why: /hello is case-insensitive
"      /Hello is case-sensitive
"      Best of both worlds

" ============================================
" USABILITY SETTINGS
" ============================================

" Enable mouse support (for beginners)
set mouse=a
" Why: Can click to position cursor
"      Useful during learning

" Show command in bottom bar
set showcmd
" Why: Shows partial commands as you type
"      Example: When you press d, shows "d" waiting for motion

" Set backspace behavior
set backspace=indent,eol,start
" Why: Backspace works intuitively
"      Can delete indents, line breaks, past start of insert

" Keep lines visible above/below cursor
set scrolloff=8
" Why: Cursor never at top/bottom of screen
"      Context always visible

" ============================================
" FILE SETTINGS
" ============================================

" Enable filetype detection
filetype on
filetype plugin on
filetype indent on
" Why: Vim recognizes file types (.py, .sh, .conf)
"      Applies appropriate syntax/indentation
```

**Save and Test:**

```bash
# After adding to ~/.vimrc:
# 1. Save the file: :wq

# 2. Test configuration:
vim test.txt

# 3. Verify:
# - See line numbers on left? ✓
# - Cursor line highlighted? ✓
# - Can see command at bottom? ✓

# 4. If not working, reload config:
:source ~/.vimrc
```

### Part 3: Plugin Management

**Installing a Plugin Manager (vim-plug):**

```bash
# Install vim-plug (most popular plugin manager)
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

# What this does:
# 1. Downloads plug.vim
# 2. Creates ~/.vim/autoload/ directory
# 3. Installs vim-plug plugin manager

# Verify installation:
ls ~/.vim/autoload/plug.vim
# Should show: /home/user/.vim/autoload/plug.vim
```

**Updated .vimrc with Plugins:**

```vim
" ============================================
" PLUGIN MANAGEMENT (using vim-plug)
" ============================================

" Initialize plugin system
call plug#begin('~/.vim/plugged')

" Plugin 1: Better syntax highlighting
Plug 'sheerun/vim-polyglot'
" Why: Supports 100+ languages
"      Better than default vim syntax

" Plugin 2: File explorer
Plug 'preservim/nerdtree'
" Why: Tree-style file browser
"      Easy navigation between files

" Plugin 3: Status line
Plug 'vim-airline/vim-airline'
" Why: Beautiful status bar
"      Shows mode, file, position, git branch

" Plugin 4: Auto-pairs (auto-close brackets)
Plug 'jiangmiao/auto-pairs'
" Why: Typing ( automatically adds )
"      Saves time, prevents errors

" Plugin 5: Fuzzy file finder
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
" Why: Quickly find files: Ctrl+P and type filename
"      Like Sublime/VSCode Ctrl+P

" Plugin 6: Git integration
Plug 'tpope/vim-fugitive'
" Why: Run git commands from vim
"      See git diff, blame, etc.

" Plugin 7: Comments
Plug 'tpope/vim-commentary'
" Why: Comment/uncomment with gc
"      Works with all file types

call plug#end()
" End plugin section

" ============================================
" PLUGIN CONFIGURATIONS
" ============================================

" NERDTree settings
map <C-n> :NERDTreeToggle<CR>
" Why: Press Ctrl+n to open/close file tree
"      Quick access to file browser

" Auto-close NERDTree when opening file
let NERDTreeQuitOnOpen=1
" Why: Tree closes after selecting file
"      Maximizes editing space

" FZF settings
map <C-p> :Files<CR>
" Why: Press Ctrl+p to search files
"      Familiar from other editors

" ============================================
" REMAINING SETTINGS FROM BASIC CONFIG
" ============================================
" (include all settings from Part 2 here)
```

**Installing Plugins:**

```bash
# 1. Save .vimrc: :wq

# 2. Open vim:
vim

# 3. Install plugins:
:PlugInstall

# What happens:
# - New window opens showing installation progress
# - Plugins download from GitHub
# - Installation takes 1-2 minutes

# Expected output:
# - Updated. Elapsed time: 0.5 sec.
# - Finishing ... Done!

# 4. Press 'q' to close installation window

# 5. Restart vim to activate plugins:
:q
vim
```

### Part 4: Color Schemes

**Installing a Color Scheme:**

```vim
" Add to plugin section of .vimrc:
call plug#begin('~/.vim/plugged')

" ... (other plugins)

" Color scheme plugins:
Plug 'morhetz/gruvbox'          " Warm, retro colors
Plug 'joshdick/onedark.vim'     " Atom One Dark
Plug 'dracula/vim', { 'as': 'dracula' }  " Dracula theme

call plug#end()

" Activate color scheme
syntax enable
set background=dark    " or 'light'
colorscheme gruvbox    " or 'onedark' or 'dracula'

" Enable 24-bit colors (if terminal supports)
set termguicolors
```

**Testing Color Schemes:**

```bash
# In vim, try different schemes:
:colorscheme gruvbox
:colorscheme onedark
:colorscheme dracula

# Find current scheme:
:colorscheme

# List available schemes:
:colorscheme <Tab>
# Press Tab repeatedly to cycle through
```

### Part 5: Complete .vimrc Template

```vim
" ============================================
" VIM CONFIGURATION - DEVOPS OPTIMIZED
" ============================================

" ============================================
" PLUGINS (vim-plug)
" ============================================
call plug#begin('~/.vim/plugged')

Plug 'sheerun/vim-polyglot'          " Better syntax
Plug 'preservim/nerdtree'            " File tree
Plug 'vim-airline/vim-airline'       " Status bar
Plug 'jiangmiao/auto-pairs'          " Auto-close brackets
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'              " Fuzzy finder
Plug 'tpope/vim-fugitive'            " Git integration
Plug 'tpope/vim-commentary'          " Easy comments
Plug 'morhetz/gruvbox'               " Color scheme

call plug#end()

" ============================================
" GENERAL SETTINGS
" ============================================
set nocompatible              " Not compatible with vi
filetype plugin indent on     " Enable file type detection
syntax on                     " Enable syntax highlighting

" ============================================
" DISPLAY
" ============================================
set number                    " Show line numbers
set relativenumber            " Show relative line numbers
set cursorline                " Highlight current line
set showmatch                 " Show matching brackets
set showcmd                   " Show command in status line
set ruler                     " Show cursor position
set laststatus=2              " Always show status line
set wildmenu                  " Enhanced command completion
set wildmode=longest:full,full " Command completion mode

" ============================================
" COLORS
" ============================================
set termguicolors             " Enable 24-bit colors
set background=dark
colorscheme gruvbox

" ============================================
" INDENTATION
" ============================================
set autoindent                " Copy indent from current line
set smartindent               " Smart auto-indenting
set expandtab                 " Tabs are spaces
set tabstop=4                 " Tab width
set shiftwidth=4              " Indent width
set softtabstop=4             " Backspace over tab

" ============================================
" SEARCH
" ============================================
set hlsearch                  " Highlight search results
set incsearch                 " Incremental search
set ignorecase                " Case-insensitive search
set smartcase                 " Case-sensitive if uppercase used

" ============================================
" USABILITY
" ============================================
set mouse=a                   " Enable mouse
set backspace=indent,eol,start " Backspace behavior
set scrolloff=8               " Keep lines above/below cursor
set encoding=utf-8            " UTF-8 encoding
set clipboard=unnamedplus     " Use system clipboard
set undofile                  " Persistent undo
set undodir=~/.vim/undo       " Undo directory
set noswapfile                " Disable swap files
set nobackup                  " Disable backup files
set splitright                " Vertical split to right
set splitbelow                " Horizontal split below

" ============================================
" KEY MAPPINGS
" ============================================
" Set leader key to space
let mapleader = " "

" NERDTree toggle
map <C-n> :NERDTreeToggle<CR>

" FZF file finder
map <C-p> :Files<CR>

" Clear search highlight
nnoremap <leader>h :nohlsearch<CR>

" Save with Ctrl+s
noremap <C-s> :w<CR>
inoremap <C-s> <Esc>:w<CR>a

" Quick quit
nnoremap <leader>q :q<CR>

" Navigate splits easily
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l

" ============================================
" FILE TYPE SPECIFIC
" ============================================
" YAML files
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 expandtab

" Python files
autocmd FileType python setlocal ts=4 sts=4 sw=4 expandtab

" Shell scripts
autocmd FileType sh setlocal ts=2 sts=2 sw=2 expandtab

" JSON files
autocmd FileType json setlocal ts=2 sts=2 sw=2 expandtab

" ============================================
" PLUGIN CONFIGURATIONS
" ============================================
" NERDTree
let NERDTreeQuitOnOpen=1      " Close after opening file
let NERDTreeShowHidden=1      " Show hidden files

" Airline
let g:airline_powerline_fonts=1
let g:airline#extensions#tabline#enabled=1
```

**Apply Configuration:**

```bash
# 1. Save .vimrc
:wq

# 2. Open new vim:
vim

# 3. Install plugins:
:PlugInstall

# 4. Create undo directory:
mkdir -p ~/.vim/undo

# 5. Verify all features:
# - Line numbers visible? ✓
# - Syntax highlighting working? ✓
# - Press Ctrl+n - NERDTree opens? ✓
# - Press Ctrl+p - File finder opens? ✓
```

---

## Multiple File Editing

### Part 1: Buffers vs Windows vs Tabs

**Understanding the Hierarchy:**

```
┌────────────────────────────────────────────────────┐
│  VIM FILE EDITING HIERARCHY:                       │
├────────────────────────────────────────────────────┤
│                                                     │
│  BUFFERS (Files in memory)                         │
│  ├─ file1.txt                                      │
│  ├─ file2.txt                                      │
│  └─ file3.txt                                      │
│      │                                              │
│      └─► WINDOWS (Visible areas)                   │
│          ├─ Window 1: Shows file1.txt              │
│          ├─ Window 2: Shows file2.txt              │
│          └─ Window 3: Shows file3.txt              │
│              │                                      │
│              └─► TABS (Workspaces)                 │
│                  ├─ Tab 1: Window layout 1         │
│                  ├─ Tab 2: Window layout 2         │
│                  └─ Tab 3: Window layout 3         │
└────────────────────────────────────────────────────┘
```

**Visual Representation:**

```
TAB 1:                          TAB 2:
┌──────────────────────┐        ┌──────────────────────┐
│    file1.txt         │        │   file4.txt          │
│                      │        ├──────────────────────┤
│ (Full window)        │        │   file5.txt          │
└──────────────────────┘        └──────────────────────┘

TAB 3:
┌───────────┬──────────┐
│ file2.txt │file3.txt │
│           │          │
│           │          │
├───────────┴──────────┤
│     file6.txt        │
└──────────────────────┘
```

### Part 2: Working with Buffers

**Opening Multiple Files:**

```bash
# From command line:
vim file1.txt file2.txt file3.txt

# Or in vim:
:e file2.txt       # Edit file2.txt (opens new buffer)
:e ../config.yml   # Edit file in different directory

# List all buffers:
:ls
# Output:
#   1 %a   "file1.txt"                    line 1
#   2      "file2.txt"                    line 0
#   3      "file3.txt"                    line 0
#
# Symbols:
# % = current buffer
# a = active (visible)
# h = hidden (not visible)
# # = alternate buffer (previous)
```

**Navigating Between Buffers:**

```bash
:bnext      # Next buffer (or :bn)
:bprevious  # Previous buffer (or :bp)
:buffer 2   # Go to buffer #2 (or :b2)
:bfirst     # First buffer (or :bf)
:blast      # Last buffer (or :bl)

# Delete buffer (close file):
:bdelete    # Delete current buffer (or :bd)
:bd 2       # Delete buffer #2

# Switch to alternate buffer:
Ctrl+6      # Toggle between current and previous buffer
            # (Very useful! Like Alt+Tab for files)
```

**Buffer Navigation Shortcuts (add to .vimrc):**

```vim
" Quick buffer navigation
nnoremap <leader>n :bnext<CR>       " Space+n = next buffer
nnoremap <leader>p :bprevious<CR>   " Space+p = previous buffer
nnoremap <leader>l :ls<CR>          " Space+l = list buffers
nnoremap <leader>d :bdelete<CR>     " Space+d = delete buffer
```

### Part 3: Window Splits

**Creating Splits:**

```bash
# Horizontal split (window above/below):
:split file.txt     # Split and open file.txt (or :sp)
:split              # Split current file
Ctrl+w s            # Split current file (shortcut)

# Vertical split (window left/right):
:vsplit file.txt    # Vertical split (or :vsp)
:vsplit             # Vertical split current file
Ctrl+w v            # Vertical split shortcut

# Example session:
vim file1.txt
:vsp file2.txt      # Vertical split
:sp file3.txt       # Horizontal split in right window

# Result:
# ┌───────────┬──────────┐
# │ file1.txt │file3.txt │
# │           ├──────────┤
# │           │file2.txt │
# └───────────┴──────────┘
```

**Navigating Between Windows:**

```bash
Ctrl+w h    # Move to window LEFT
Ctrl+w j    # Move to window DOWN
Ctrl+w k    # Move to window UP
Ctrl+w l    # Move to window RIGHT

Ctrl+w w    # Cycle through windows
Ctrl+w q    # Close current window

# Resize windows:
Ctrl+w =    # Make all windows equal size
Ctrl+w _    # Maximize current window height
Ctrl+w |    # Maximize current window width

Ctrl+w +    # Increase height
Ctrl+w -    # Decrease height
Ctrl+w >    # Increase width
Ctrl+w <    # Decrease width

# With number prefix:
10 Ctrl+w + # Increase height by 10 lines
```

**Window Navigation Shortcuts (add to .vimrc):**

```vim
" Easier window navigation (no Ctrl+w needed)
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l

" Now: Ctrl+h moves left, Ctrl+j moves down, etc.
```

### Part 4: Tabs

**Creating and Managing Tabs:**

```bash
# Create new tab:
:tabnew             # Empty tab
:tabnew file.txt    # Tab with file
:tabe file.txt      # Tab edit (shorter)

# From command line:
vim -p file1.txt file2.txt file3.txt
# Opens each file in separate tab

# Navigate tabs:
:tabnext    # Next tab (or :tabn, or gt)
:tabprev    # Previous tab (or :tabp, or gT)
:tabfirst   # First tab (or :tabfir)
:tablast    # Last tab (or :tabl)

# Go to specific tab:
1gt         # Go to tab 1
2gt         # Go to tab 2
3gt         # Go to tab 3

# Close tab:
:tabclose   # Close current tab (or :tabc)
:tabonly    # Close all tabs except current (or :tabo)
```

**Tab Shortcuts (add to .vimrc):**

```vim
" Tab navigation
nnoremap <leader>tn :tabnew<CR>      " Space+tn = new tab
nnoremap <leader>tc :tabclose<CR>    " Space+tc = close tab
nnoremap <leader>1 1gt               " Space+1 = tab 1
nnoremap <leader>2 2gt               " Space+2 = tab 2
nnoremap <leader>3 3gt               " Space+3 = tab 3
```

**Tabs vs Splits - When to Use:**

```
┌────────────────────────────────────────────────────┐
│  USE TABS WHEN:                                    │
│  ✓ Working on different logical tasks             │
│  ✓ Need complete context switch                   │
│  ✓ Different projects/config groups                │
│                                                     │
│  Example:                                          │
│  Tab 1: Nginx config (nginx.conf, sites-enabled/) │
│  Tab 2: Application code (app.py, config.py)      │
│  Tab 3: Documentation (README.md, NOTES.txt)       │
├────────────────────────────────────────────────────┤
│  USE SPLITS WHEN:                                  │
│  ✓ Need to see multiple files simultaneously      │
│  ✓ Comparing files side-by-side                   │
│  ✓ Reference while editing                        │
│                                                     │
│  Example:                                          │
│  Left: nginx.conf (editing)                        │
│  Right: nginx.conf.backup (reference)              │
└────────────────────────────────────────────────────┘
```

### Part 5: Practical Multi-File Workflow

**Scenario: Edit Multiple Config Files**

```bash
# Task: Update configuration across multiple files
# Files: /etc/nginx/nginx.conf, /etc/nginx/sites-available/default, /etc/php/php.ini

# Method 1: Using tabs
vim -p /etc/nginx/nginx.conf /etc/nginx/sites-available/default /etc/php/php.ini

# Now in vim:
# Tab 1: nginx.conf
# Tab 2: default
# Tab 3: php.ini

# Navigate: gt (next tab), gT (previous tab)
# Edit each file, save with :w
# Close all: :wqa (write all, quit all)

# Method 2: Using splits
vim /etc/nginx/nginx.conf
:vsp /etc/nginx/sites-available/default
:sp /etc/php/php.ini

# Result:
# ┌──────────────┬────────────┐
# │ nginx.conf   │   php.ini  │
# │              ├────────────┤
# │              │  default   │
# └──────────────┴────────────┘

# Navigate: Ctrl+h, Ctrl+j, Ctrl+k, Ctrl+l
# Save all: :wa
# Quit all: :qa
```

**Scenario: Find and Replace Across Files**

```bash
# Task: Change "port 8080" to "port 8090" in all config files

# Open all files in buffers:
vim *.conf

# Show all buffers:
:ls

# Run command on all buffers:
:bufdo %s/port 8080/port 8090/ge | update
# Breakdown:
# :bufdo        → Execute command on all buffers
# %s/8080/8090/ → Search and replace
# g             → Global (all occurrences)
# e             → No error if pattern not found
# | update      → Save each buffer after change

# Verify changes:
:bnext    # Check each file
```

---

## System Config File Editing

### Part 1: Common System Config Files

**File Locations Reference:**

```
┌────────────────────────────────────────────────────┐
│  CRITICAL SYSTEM CONFIG FILES:                     │
├────────────────────────────────────────────────────┤
│  /etc/nginx/nginx.conf          → Nginx main config│
│  /etc/nginx/sites-available/*   → Nginx vhosts    │
│  /etc/apache2/apache2.conf      → Apache config   │
│  /etc/ssh/sshd_config            → SSH daemon     │
│  /etc/hosts                      → Host mapping   │
│  /etc/fstab                      → Filesystem mounts│
│  /etc/systemd/system/*.service   → Systemd units  │
│  /etc/environment                → Environment vars│
│  ~/.bashrc                       → Bash config    │
│  ~/.ssh/config                   → SSH client     │
└────────────────────────────────────────────────────┘
```

### Part 2: Safe Editing Practices

**Always Backup Before Editing:**

```bash
# Create backup with timestamp
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup.$(date +%Y%m%d-%H%M%S)

# Verify backup:
ls -l /etc/nginx/*.backup.*

# Open original for editing:
sudo vim /etc/nginx/nginx.conf
```

**Editing Workflow:**

```bash
# 1. Backup
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# 2. Edit
sudo vim /etc/ssh/sshd_config

# 3. Inside vim, before making changes:
:w /tmp/sshd_config.test    # Save test copy
:e /tmp/sshd_config.test    # Edit test copy

# 4. Make changes in test copy
# ... edit ...

# 5. Validate syntax (if tool exists):
# For sshd_config:
:!sudo sshd -t -f /tmp/sshd_config.test

# For nginx:
:!sudo nginx -t -c /tmp/nginx.conf.test

# 6. If validation passes, copy to real location:
:!sudo cp /tmp/sshd_config.test /etc/ssh/sshd_config

# 7. Restart service:
:!sudo systemctl restart sshd
```

### Part 3: Vim as Root vs Sudo Vim

**Security Considerations:**

```bash
# ❌ UNSAFE: Opening vim as root
sudo vim /etc/nginx/nginx.conf
# Risk: Plugins run as root, .vimrc executed as root

# ✅ SAFER: Edit as user, write with sudo
vim /etc/nginx/nginx.conf
# Make changes...
# When saving:
:w !sudo tee %
# Breakdown:
# :w        → Write command
# !sudo tee → Pipe content to sudo tee
# %         → Current filename

# Or add to .vimrc:
" Allow saving files with sudo
cmap w!! w !sudo tee % > /dev/null

# Then save with:
:w!!
```

### Part 4: Real-World Editing Scenarios

#### Scenario 1: Edit Nginx Configuration

```bash
# Task: Add new server block to nginx

# 1. Backup current config
sudo cp /etc/nginx/sites-available/default \
        /etc/nginx/sites-available/default.backup

# 2. Open vim
sudo vim /etc/nginx/sites-available/default

# 3. Navigate to end of file
G

# 4. Add new server block
o    # Open new line below
# Press i to enter INSERT mode
# Type new server block:

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
    }
}

# 5. Save and test
:w                              # Save
:!sudo nginx -t                 # Test config
# Output: nginx: configuration file /etc/nginx/nginx.conf test is successful

# 6. Reload nginx
:!sudo systemctl reload nginx

# 7. Verify
:!curl -I http://example.com
```

#### Scenario 2: Edit Multiple SSH Configs

```bash
# Task: Update SSH settings on multiple servers

# Open files in tabs:
vim -p \
    /etc/ssh/sshd_config \
    ~/.ssh/config \
    ~/.ssh/authorized_keys

# Tab 1: /etc/ssh/sshd_config
# Search for PermitRootLogin:
/PermitRootLogin
n    # Next match
cw   # Change word
no   # Type "no"
Esc  # Exit INSERT mode

# Tab 2: ~/.ssh/config
gt   # Next tab
# Add new host config:
G    # End of file
o    # New line
Host production
    HostName 192.168.1.100
    User deploy
    IdentityFile ~/.ssh/production_key
Esc

# Tab 3: ~/.ssh/authorized_keys
gt   # Next tab
# Find and remove old key:
/old-key-comment
dd   # Delete line

# Save all tabs:
:wqa
```

#### Scenario 3: Systemd Service File

```bash
# Task: Create new systemd service

# Create file:
sudo vim /etc/systemd/system/myapp.service

# Template (type in INSERT mode):
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=myuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
Restart=always

[Install]
WantedBy=multi-user.target

# Save and enable:
:w
:!sudo systemctl daemon-reload
:!sudo systemctl enable myapp.service
:!sudo systemctl start myapp.service
:!sudo systemctl status myapp.service
```

### Part 5: Advanced Editing Techniques

**Search and Replace in Config Files:**

```bash
# Example: Change all instances of port 8080 to 8090

# Open file:
vim /etc/nginx/nginx.conf

# Preview changes (dry run):
:%s/8080/8090/gn
# n = show count, don't replace

# Replace with confirmation:
:%s/8080/8090/gc
# For each match:
# replace with 8090 (y/n/a/q/l/^E/^Y)?
# y = yes
# n = no
# a = all remaining
# q = quit
# l = this one and quit
```

**Visual Block Mode (Multi-line Editing):**

```bash
# Task: Comment out multiple lines

# Navigate to first line:
/listen
# Press Ctrl+v (enter VISUAL BLOCK mode)
# Move down with j to select multiple lines
jjjj  # Select 5 lines

# Insert # at start of each line:
I     # Insert at start of block
#     # Type comment character
Esc   # Apply to all selected lines

# Result:
# listen 80;
# listen 443;
# listen 8080;
# ...

# Uncomment (remove first character):
Ctrl+v    # Visual block
jjjj      # Select lines
x         # Delete first character of each
```

**Macros for Repetitive Edits:**

```bash
# Task: Format multiple similar lines

# Position cursor on first line
# Start recording macro into register 'a':
qa

# Perform actions:
I    # Insert at start
#    # Add comment
Esc  # Exit insert
j    # Move down
^    # Go to first non-whitespace

# Stop recording:
q

# Replay macro:
@a   # Once
5@a  # 5 times
```

---

## Muscle Memory Drills

### Drill 1: Navigation Speed Test (5 minutes)

```bash
# Setup:
# Create test file with 100 lines:
seq 1 100 | xargs -I {} echo "Line {}: Lorem ipsum dolor sit amet" > drill1.txt

# Drill:
vim drill1.txt

# Tasks (time yourself):
# 1. Go to line 50 (2 seconds)
50G

# 2. Go to end of file (1 second)
G

# 3. Go to beginning (1 second)
gg

# 4. Search for "50" (3 seconds)
/50
n   # Next match

# 5. Go to line 25, delete to line 75 (5 seconds)
25G
d50j   # Delete current + 50 lines down

# Goal: Complete all in under 15 seconds
# Repeat 10 times
```

### Drill 2: Insertion Mode Mastery (5 minutes)

```bash
# Setup:
echo "Hello World" > drill2.txt
vim drill2.txt

# Tasks (repeat each 10 times):

# Task 1: Insert at cursor
0    # Start of line
i    # Insert mode
Testing
Esc  # Normal mode
u    # Undo

# Task 2: Append after cursor
$    # End of line
a    # Append mode
Testing
Esc
u

# Task 3: Insert at line start
I    # Insert at line start (shortcut for ^i)
Testing
Esc
u

# Task 4: Append at line end
A    # Append at line end (shortcut for $a)
Testing
Esc
u

# Task 5: New line below
o    # New line below
Testing
Esc
dd   # Delete line

# Task 6: New line above
O    # New line above
Testing
Esc
dd

# Goal: Each action without thinking, < 1 second
```

### Drill 3: Delete and Change Operations (10 minutes)

```bash
# Setup:
cat > drill3.txt << EOF
The quick brown fox jumps over the lazy dog.
Pack my box with five dozen liquor jugs.
How vexingly quick daft zebras jump!
EOF

vim drill3.txt

# Practice each command 5 times:

# 1. Delete word
w    # Move to start of word
dw   # Delete word
u    # Undo

# 2. Delete to end of line
$    # End of line
d0   # Delete to start
u

# 3. Delete entire line
dd
u

# 4. Change word (delete and insert)
cw   # Change word
NewWord
Esc
u

# 5. Change to end of line
c$   # Change to end
New text
Esc
u

# 6. Delete 3 words
d3w
u

# 7. Change 2 words
c2w
New words
Esc
u

# Goal: Execute each without hesitation
```

### Drill 4: Copy and Paste (10 minutes)

```bash
# Setup:
cat > drill4.txt << EOF
Line 1
Line 2
Line 3
Line 4
Line 5
EOF

vim drill4.txt

# Tasks:

# 1. Copy (yank) line
yy   # Yank line
p    # Paste below
u

# 2. Copy word
yw   # Yank word
p    # Paste
u

# 3. Copy 3 lines
y3j  # Yank current + 3 down
p
u

# 4. Cut (delete) and paste
dd   # Delete (cut) line
p    # Paste

# 5. Visual mode copy
V    # Visual line mode
jj   # Select 3 lines
y    # Yank
p    # Paste
u

# Goal: Smooth execution, no thinking
```

### Drill 5: Search and Replace Mastery (10 minutes)

```bash
# Setup:
cat > drill5.txt << EOF
server {
    listen 8080;
    server_name example.com;
    location / {
        proxy_pass http://localhost:8080;
        proxy_port 8080;
    }
}
EOF

vim drill5.txt

# Tasks:

# 1. Find all instances of 8080
/8080
# Press n repeatedly to cycle through

# 2. Replace first on current line
:s/8080/9090/
# Result: Changes first 8080 on current line
u

# 3. Replace all on current line
:s/8080/9090/g
u

# 4. Replace all in file
:%s/8080/9090/g
u

# 5. Replace with confirmation
:%s/8080/9090/gc
# Press y for each

# 6. Case-insensitive search
/EXAMPLE\c
# \c = case insensitive

# Goal: Know each variant without reference
```

### Drill 6: Multi-File Circuit (15 minutes)

```bash
# Setup:
mkdir -p ~/vim-drill
cd ~/vim-drill
echo "File 1 content" > file1.txt
echo "File 2 content" > file2.txt
echo "File 3 content" > file3.txt

# Circuit:
vim file1.txt file2.txt file3.txt

# Task 1: Navigate buffers (30 seconds)
:bnext
:bnext
:bprev
:b1

# Task 2: Create splits (30 seconds)
:vsp file2.txt
Ctrl+w v    # Another split
:e file3.txt

# Task 3: Navigate splits (30 seconds)
Ctrl+w h
Ctrl+w l
Ctrl+w j
Ctrl+w k

# Task 4: Create tabs (30 seconds)
:tabnew file1.txt
:tabnew file2.txt
gt
gT
1gt

# Task 5: Edit across files (2 minutes)
# In file1.txt:
iNew content
Esc
:bnext
# In file2.txt:
iMore content
Esc
:w
:bp

# Task 6: Save all and quit (10 seconds)
:wa   # Write all
:qa   # Quit all

# Total time goal: < 5 minutes
# Repeat until automatic
```

---

## Real-World DevOps Scenarios

### Scenario 1: Emergency Log Analysis

**Situation:** Production server error, need to analyze logs quickly

```bash
# Open massive log file
vim /var/log/nginx/error.log

# File is huge (10,000+ lines), scrolling slow

# Task 1: Jump to end (most recent errors)
G    # End of file

# Task 2: Search for "500" errors
?500    # Search backwards
N       # Previous match (going backwards through errors)

# Task 3: Find errors in last hour
# Current time: 15:30
/15:[0-9][0-9]    # Regex for 15:XX timestamps
n                 # Next match

# Task 4: Copy error context
# Position on error line
V     # Visual line mode
5j    # Select 5 lines
y     # Yank

# Task 5: Paste into new file for analysis
:tabnew /tmp/errors.txt
p     # Paste

# Task 6: Filter unique errors
:%!sort | uniq
# % = entire file
# ! = pipe to shell command
# sort | uniq = remove duplicates

# Result: Unique errors identified in under 1 minute
```

### Scenario 2: Configuration Rollout

**Situation:** Update nginx config across multiple sites

```bash
# Open all site configs
vim /etc/nginx/sites-available/*

# Files opened as buffers
:ls
# Shows 10+ buffers

# Task: Add security header to all sites

# Method 1: bufdo (command across all buffers)
:bufdo /location \//
:bufdo normal o        add_header X-Frame-Options "SAMEORIGIN";
:bufdo normal o        add_header X-Content-Type-Options "nosniff";
# Breakdown:
# :bufdo = execute in all buffers
# /location \// = search for "location /"
# normal o = execute "o" in normal mode (new line below)
# Then type header lines

# Save all:
:wa

# Validate all:
:!sudo nginx -t

# If success:
:!sudo systemctl reload nginx
```

### Scenario 3: Multi-Server Config Sync

**Situation:** Edit local config, then sync to multiple servers

```bash
# Edit local copy
vim ~/configs/app.conf

# Make changes...
# Save: :w

# Deploy to servers (without leaving vim)
:!scp % user@server1:/etc/app/app.conf
:!scp % user@server2:/etc/app/app.conf
:!scp % user@server3:/etc/app/app.conf

# % = current filename

# Restart services remotely
:!ssh user@server1 'sudo systemctl restart app'
:!ssh user@server2 'sudo systemctl restart app'
:!ssh user@server3 'sudo systemctl restart app'

# Verify
:!ssh user@server1 'systemctl status app'
```

### Scenario 4: Dockerfile Creation

**Situation:** Create and test Dockerfile

```bash
# Create Dockerfile
vim Dockerfile

# Enter content:
i
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y nginx
COPY config/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
Esc

# Save: :w

# Build image (without leaving vim)
:!docker build -t myapp:latest .

# View build output in vim:
:r !docker build -t myapp:latest .
# Output inserted into file

# Test run:
:!docker run -d -p 80:80 myapp:latest

# Check logs:
:!docker logs $(docker ps -q -n 1)
```

### Scenario 5: Automated Configuration Template

**Situation:** Generate config files for multiple environments

```bash
# Create template
vim template.conf

# Base configuration:
server {
    listen 80;
    server_name ENV_HOSTNAME;
    root /var/www/ENV_NAME;
    
    location / {
        proxy_pass http://ENV_BACKEND;
    }
}

# Save template: :w

# Generate production config:
:%s/ENV_HOSTNAME/prod.example.com/g
:%s/ENV_NAME/production/g
:%s/ENV_BACKEND/10.0.1.100:8080/g
:w prod.conf

# Reload template:
:e template.conf

# Generate staging config:
:%s/ENV_HOSTNAME/staging.example.com/g
:%s/ENV_NAME/staging/g
:%s/ENV_BACKEND/10.0.2.100:8080/g
:w staging.conf

# Better: Use macro
# Record once:
qa                                          # Start recording
:%s/ENV_HOSTNAME/HOSTNAME/g<CR>
:call inputsave()<CR>
:let host=input('Hostname: ')<CR>
:call inputrestore()<CR>
:%s/HOSTNAME/<C-r>=host<CR>/g<CR>
:w <C-r>=host<CR>.conf<CR>
:e template.conf<CR>
q                                           # Stop recording

# Now generate any environment:
@a
# Prompts for hostname, generates file
```

### Scenario 6: Log Parsing and Reporting

**Situation:** Extract specific data from logs

```bash
# Open access log
vim /var/log/nginx/access.log

# Find all 404 errors
:g/404/
# :g = global command (on matching lines)

# Copy 404 lines to new file
:g/404/t$
# g = global
# 404 = pattern
# t = copy (t for "transfer")
# $ = to end of file

# Or write to separate file:
:g/404/w >> /tmp/404-errors.log

# Count occurrences:
:%s/404//gn
# Result: "23 matches on 23 lines"

# Extract IPs with 404s:
:v/404/d     # Delete lines NOT containing 404
:%s/^\(.*\) -.*/\1/    # Extract IP (first field)
:%!sort | uniq -c | sort -rn    # Count and sort

# Result: List of IPs by 404 count
```

---

## Advanced Techniques

### Technique 1: Vim Registers (Clipboard Management)

```bash
# Vim has 26 named registers (a-z)

# Yank to specific register:
"ayy    # Yank line to register a
"byw    # Yank word to register b

# Paste from specific register:
"ap     # Paste from register a
"bp     # Paste from register b

# View all registers:
:reg

# Special registers:
" (unnamed)  - Last yank/delete
0 (zero)     - Last yank only
+ (plus)     - System clipboard
* (star)     - Selection clipboard

# Copy to system clipboard:
"+yy    # Copy line to system clipboard
"+p     # Paste from system clipboard

# Use case:
# Copy from vim, paste into browser:
"+yy
# Copy from browser, paste into vim:
"+p
```

### Technique 2: Advanced Search Patterns

```bash
# Regex patterns:

# Find lines starting with #
/^#

# Find lines ending with ;
/;$

# Find IP addresses
/\d\+\.\d\+\.\d\+\.\d\+

# Find email addresses
/\w\+@\w\+\.\w\+

# Find URLs
/https\?:\/\/[^ ]\+

# Case-insensitive search
/pattern\c

# Case-sensitive search (even with ignorecase set)
/pattern\C

# Search for word under cursor
*    # Forward
#    # Backward
```

### Technique 3: Vim Scripting (Automation)

```vim
" Add to .vimrc for custom functions

" Function: Insert timestamp
function! InsertTimestamp()
    let timestamp = strftime("%Y-%m-%d %H:%M:%S")
    execute "normal! i" . timestamp
endfunction

" Map to key:
nnoremap <leader>ts :call InsertTimestamp()<CR>

" Function: Auto-format JSON
function! FormatJSON()
    :%!python -m json.tool
endfunction

command! JsonFormat call FormatJSON()

" Usage: :JsonFormat

" Function: Strip trailing whitespace
function! StripTrailingWhitespace()
    let save_cursor = getpos(".")
    %s/\s\+$//e
    call setpos('.', save_cursor)
endfunction

" Auto-strip on save:
autocmd BufWritePre * :call StripTrailingWhitespace()
```

### Technique 4: Vim Sessions

```bash
# Save current session:
:mksession ~/work-session.vim

# What's saved:
# - All open files
# - Window layout
# - Tabs
# - Cursor positions

# Restore session:
vim -S ~/work-session.vim

# Or from within vim:
:source ~/work-session.vim

# Update existing session:
:mksession! ~/work-session.vim
```

### Technique 5: Diff Mode

```bash
# Compare two files:
vim -d file1.txt file2.txt

# Or from within vim:
:vertical diffsplit file2.txt

# Navigation:
]c    # Next change
[c    # Previous change

# Merge changes:
do    # Diff obtain (get change from other file)
dp    # Diff put (put change to other file)

# Update diff:
:diffupdate

# Exit diff mode:
:diffoff
```

---

## Cheat Sheet

### Essential Commands

```
┌────────────────────────────────────────────────────┐
│  MOVEMENT                                          │
├────────────────────────────────────────────────────┤
│  h j k l        → Left, Down, Up, Right            │
│  w b            → Word forward/backward            │
│  0 $            → Start/end of line                │
│  gg G           → Start/end of file                │
│  50G            → Go to line 50                    │
│  %              → Jump to matching bracket         │
├────────────────────────────────────────────────────┤
│  EDITING                                           │
├────────────────────────────────────────────────────┤
│  i a o          → Insert, Append, Open line        │
│  I A O          → Insert start, Append end, Open above│
│  x              → Delete character                 │
│  dw dd d$       → Delete word, line, to end        │
│  cw cc c$       → Change word, line, to end        │
│  yy             → Yank (copy) line                 │
│  p P            → Paste after/before               │
│  u Ctrl+r       → Undo, Redo                       │
├────────────────────────────────────────────────────┤
│  SEARCH                                            │
├────────────────────────────────────────────────────┤
│  /pattern       → Search forward                   │
│  ?pattern       → Search backward                  │
│  n N            → Next, Previous match             │
│  *              → Search word under cursor         │
├────────────────────────────────────────────────────┤
│  REPLACE                                           │
├────────────────────────────────────────────────────┤
│  :s/old/new/    → Replace on line                  │
│  :%s/old/new/g  → Replace in file                  │
│  :%s/old/new/gc → Replace with confirmation        │
├────────────────────────────────────────────────────┤
│  FILES                                             │
├────────────────────────────────────────────────────┤
│  :w             → Save                             │
│  :q             → Quit                             │
│  :wq            → Save and quit                    │
│  :q!            → Quit without saving              │
│  :e file        → Edit file                        │
├────────────────────────────────────────────────────┤
│  BUFFERS                                           │
├────────────────────────────────────────────────────┤
│  :ls            → List buffers                     │
│  :bnext :bprev  → Next/previous buffer             │
│  :b2            → Go to buffer 2                   │
│  :bd            → Delete buffer                    │
├────────────────────────────────────────────────────┤
│  WINDOWS                                           │
├────────────────────────────────────────────────────┤
│  :split :vsplit → Horizontal/vertical split        │
│  Ctrl+w hjkl    → Navigate windows                 │
│  Ctrl+w q       → Close window                     │
│  Ctrl+w =       → Equalize windows                 │
├────────────────────────────────────────────────────┤
│  TABS                                              │
├────────────────────────────────────────────────────┤
│  :tabnew        → New tab                          │
│  gt gT          → Next/previous tab                │
│  :tabclose      → Close tab                        │
└────────────────────────────────────────────────────┘
```

---

## Final Mastery Test (30 minutes)

```bash
# Setup:
mkdir -p ~/vim-test
cd ~/vim-test

# Create test files:
cat > server1.conf << EOF
server {
    listen 8080;
    server_name server1.local;
    root /var/www/server1;
}
EOF

cat > server2.conf << EOF
server {
    listen 8080;
    server_name server2.local;
    root /var/www/server2;
}
EOF

cat > app.log << EOF
2024-01-15 10:00:00 INFO Starting application
2024-01-15 10:01:00 ERROR Database connection failed
2024-01-15 10:02:00 INFO Retrying connection
2024-01-15 10:03:00 ERROR Timeout
2024-01-15 10:04:00 INFO Application started
EOF

# TEST TASKS:

# Task 1 (3 min): Edit multiple configs
# Open both conf files in tabs
# Change port from 8080 to 9090 in both
# Save both files
vim -p server1.conf server2.conf
# In vim:
:%s/8080/9090/g
gt
:%s/8080/9090/g
:wqa

# Task 2 (5 min): Log analysis
# Open app.log
# Extract all ERROR lines to new file
# Count number of errors
vim app.log
:g/ERROR/w >> errors.txt
:%s/ERROR//gn    # Should show count

# Task 3 (5 min): Create from template
# Create template.conf with ENV_ placeholders
# Generate 3 environment configs
vim template.conf
# Create dev, staging, prod versions

# Task 4 (4 min): Multi-window editing
# Open server1.conf and server2.conf in splits
# Copy server block from server1 to server2
vim server1.conf
:vsp server2.conf
# Visual select and yank
# Switch window and paste

# Task 5 (3 min): Search and replace across files
# Change all "server" to "nginx" in all .conf files
vim *.conf
:bufdo %s/server/nginx/ge | update

# Task 6 (2 min): Macro recording
# Record macro to comment line and move down
# Apply to 5 lines
qa
I#
Esc
j
q
5@a

# Task 7 (3 min): External command integration
# Insert current date into file
# Run system command from vim
:r !date
:!ls -la

# Task 8 (5 min): Emergency fix simulation
# Backup file, edit, validate, restore if needed
:!cp server1.conf server1.conf.bak
# Edit...
:!diff server1.conf server1.conf.bak
# If wrong:
:!mv server1.conf.bak server1.conf

# SCORING:
# < 20 min: Master
# 20-25 min: Advanced
# 25-30 min: Proficient
# > 30 min: Need more practice
```

---

## Daily Practice Routine

```
┌────────────────────────────────────────────────────┐
│  15-MINUTE DAILY VIM PRACTICE                      │
├────────────────────────────────────────────────────┤
│  Day 1-7: Vimtutor + Basic Navigation              │
│    - Run vimtutor (15 min)                         │
│    - Focus on hjkl, not arrow keys                 │
│                                                     │
│  Day 8-14: Editing Operations                      │
│    - Drills 1-3 (5 min each)                       │
│    - Real file editing practice                    │
│                                                     │
│  Day 15-21: Multi-file Workflows                   │
│    - Buffers, windows, tabs practice               │
│    - Edit actual project files                     │
│                                                     │
│  Day 22-28: Advanced Techniques                    │
│    - Macros, registers, search/replace             │
│    - Custom .vimrc additions                       │
│                                                     │
│  Day 29+: Real-World Application                   │
│    - All config editing in vim only                │
│    - Build custom workflows                        │
│    - Teach others (best solidification)            │
└────────────────────────────────────────────────────┘
```

**Pro Tips:**
1. **No arrow keys!** Force yourself to use hjkl
2. **No mouse!** Learn keyboard shortcuts
3. **One new command per day** - master it completely
4. **Edit real files** - Practice > Theory
5. **vimtutor is magic** - Run it regularly

Your hands will remember. Trust the process! 🚀