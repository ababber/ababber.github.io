---
title: How I Improved My Shell Load Time With a Lazy Load
date: 2025-02-17
draft: false 
tags:
  - bash
  - zsh
  - nvm
  - node
  - npm
  - time
  - zmodload
  - zprof
  - lazy-load
---

> If you've used multiple tools, languages, build environments, etc.. as a CLI (command line utility), then you've probably experienced slow load times for your shell environment at some point. A simple way to determine which CLI is the slowest is to measure startup time. After identifying the slowest CLI tool with a profile, lazy loading that tool to the environment is the simplest approach to try when attempting to improve load times for a shell environment. I will provide an example specific to lazy loading `zsh`, but the general idea can be applied to other environments.

## 0. Diagnosing the problem

```sh
time zsh -i -c exit
```

* When my shell starts feeling a little sluggish, I use `time` to get an accurate measure of what I'm feeling.
* On `unix` systems with a shell, you'll probably have some version of the [`time`](https://www.gnu.org/software/time/) utility. To get the specific information for your specific shell, I recommend reading `man time`. The utility is useful for measuring CLI program run times.
* In our example, we are simply passing `zsh` to `time` to see how long it takes for it to start and stop.
* The `-i` flag forces `zsh` to be in interactive mode.
* The `-c` flag passes any commands to `zsh` in interactive mode. Remember to use native tools or tools in your `PATH`.
* Output:

```TEXT
Saving session...completed.
zsh -i -c exit  0.66s user 0.72s system 89% cpu 1.543 total
```

* At first the results don't seem so bad, but we can get a more granular look at what is being loaded.
* In your `.zshrc` add the following:

```bash
# beginning of .zshrc
zmodload zsh/zprof

# contents of .zshrc must be in between

# end of .zshrc
zprof
```

* [zmodload](https://zsh.sourceforge.io/Doc/Release/Shell-Builtin-Commands.html#index-zmodload) loads the built in `zsh` modules on launch if added to the `.zshrc` as above.
* The [zsh/zprof](https://zsh.sourceforge.io/Doc/Release/Zsh-Modules.html#The-zsh_002fzprof-Module) module profiles everything that is loaded on start in `zsh`.
* On startup, `zsh` will output something similar to the output below. Once the source of the slow load time is identified, I usually comment out `zmodload zsh/zprof` and `zprof` in my `.zshrc` for future use.
* Output:

```TEXT
num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)    1         835.86   835.86   80.69%    309.23   309.23   29.85%  nvm_auto
 2)    2         480.08   240.04   46.35%    278.23   139.12   26.86%  nvm
 3)    1         170.17   170.17   16.43%    145.25   145.25   14.02%  nvm_ensure_version_installed
 4)   26         104.45     4.02   10.08%     78.22     3.01    7.55%  _omz_source
 5)    4          58.36    14.59    5.63%     58.36    14.59    5.63%  compaudit
 6)    1          31.50    31.50    3.04%     31.29    31.29    3.02%  nvm_die_on_prefix
 7)    2          87.03    43.51    8.40%     28.66    14.33    2.77%  compinit
 8)    1          46.55    46.55    4.49%     25.39    25.39    2.45%  nvm_is_valid_version
 9)    1          24.92    24.92    2.41%     24.92    24.92    2.41%  nvm_is_version_installed
10)    1          15.93    15.93    1.54%     11.14    11.14    1.08%  nvm_validate_implicit_alias
11)    1          10.13    10.13    0.98%      9.98     9.98    0.96%  _zsh_highlight_load_highlighters
12)    1           7.96     7.96    0.77%      7.96     7.96    0.77%  zrecompile
13)    1           5.81     5.81    0.56%      5.81     5.81    0.56%  test-ls-args
14)    1           5.23     5.23    0.51%      5.23     5.23    0.51%  nvm_version_greater_than_or_equal_to
15)    1           4.72     4.72    0.46%      4.72     4.72    0.46%  nvm_echo
16)    1           2.12     2.12    0.21%      2.10     2.10    0.20%  _zsh_highlight__function_callable_p
17)    1           1.67     1.67    0.16%      1.67     1.67    0.16%  regexp-replace
18)    3           1.48     0.49    0.14%      1.40     0.47    0.14%  add-zle-hook-widget
19)    9           1.37     0.15    0.13%      1.37     0.15    0.13%  is-at-least
20)   13           1.35     0.10    0.13%      1.35     0.10    0.13%  compdef
21)    1           1.16     1.16    0.11%      1.16     1.16    0.11%  colors
22)    7           0.92     0.13    0.09%      0.92     0.13    0.09%  add-zsh-hook
23)    1           0.27     0.27    0.03%      0.27     0.27    0.03%  (anon) [/Users/ankit/.oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh:458]
24)    4           0.21     0.05    0.02%      0.21     0.05    0.02%  nvm_npmrc_bad_news_bears
25)    6           0.18     0.03    0.02%      0.18     0.03    0.02%  is_plugin
26)    1           0.17     0.17    0.02%      0.17     0.17    0.02%  nvm_has
27)    1           0.13     0.13    0.01%      0.13     0.13    0.01%  zvm_exist_command
28)    1           0.22     0.22    0.02%      0.10     0.10    0.01%  complete
29)    3           0.09     0.03    0.01%      0.09     0.03    0.01%  is_theme
30)    1           0.08     0.08    0.01%      0.08     0.08    0.01%  (anon) [/usr/local/Cellar/zsh/5.9/share/zsh/functions/add-zle-hook-widget:28]
31)    2           0.07     0.04    0.01%      0.07     0.04    0.01%  env_default
32)    1           4.79     4.79    0.46%      0.07     0.07    0.01%  nvm_err
33)    2           0.07     0.03    0.01%      0.07     0.03    0.01%  bashcompinit
34)    1         835.90   835.90   80.69%      0.04     0.04    0.00%  nvm_process_parameters
35)    1           0.02     0.02    0.00%      0.02     0.02    0.00%  _zsh_highlight__is_function_p
36)    1           0.01     0.01    0.00%      0.01     0.01    0.00%  nvm_is_zsh
37)    1           0.00     0.00    0.00%      0.00     0.00    0.00%  _zsh_highlight_bind_widgets
```

* As a reminder, whenever `.zshrc` is changed. Saving then reloading it will ensure all changes are reflected. Reload with `source .PATH/TO/.zshrc`.

## 1. Lazy Load

* From the output of `zmodload zsh/zprof`, it seems `nvm` is the likely reason my start up time is slow. If I lazy load the right things, I could improve my `zsh` start up time.
* Lazy loading depends on your environment. If your local configuration is slightly different, then focusing on where your tools are loaded from is a reasonable starting point. Delaying the load of the slowest tool is the goal.
* To lazy load `nvm`, I added wrapper functions around the `nvm`, `node`, `npm`, and `npx` variable names to change their respective scope (a.k.a overloading variable name). `zsh` will see the function before something further down the `PATH` chain. When `nvm`, `node`, `npm`, or `npx` is explicitly called then `zsh` will load the resources into the current session.
* For me, adding the below functions to my `.zshrc` fixed my `zsh` load time.

```bash
# lazy lode nvm instead of through oh-my-zsh to reduce load by 50%
lazy-nvm()
{
  unset -f nvm node npm npx
  export NVM_DIR="$HOME/.nvm"
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
  [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
}

nvm()
{
  lazy-nvm
  nvm $@
}

node()
{
  lazy-nvm
  node $@
}

npm()
{
  lazy-nvm
  npm $@
}

npx()
{
  lazy-nvm
  npx $@
}
```

## 2. Verification

```sh
time zsh -i -c exit
```

* After lazy loading `nvm`, my `zsh` startup time was cut in half.

```TEXT
Saving session...completed.
zsh -i -c exit  0.32s user 0.31s system 92% cpu 0.673 total
```

* Adding the below lines to the `.zshrc` again will confirm `nvm` isn't loaded on start:

```bash
# beginning of .zshrc
zmodload zsh/zprof

# contents of .zshrc must be in between

# end of .zshrc
zprof
```

* Output:

```TEXT
num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)   26         109.34     4.21   71.45%     82.96     3.19   54.22%  _omz_source
 2)    2          29.39    14.70   19.21%     29.39    14.70   19.21%  compaudit
 3)    1          10.67    10.67    6.97%     10.51    10.51    6.87%  _zsh_highlight_load_highlighters
 4)    1          36.67    36.67   23.96%      7.27     7.27    4.75%  compinit
 5)    1           6.67     6.67    4.36%      6.67     6.67    4.36%  zrecompile
 6)    1           5.15     5.15    3.36%      5.15     5.15    3.36%  test-ls-args
 7)    1           2.21     2.21    1.45%      2.19     2.19    1.43%  _zsh_highlight__function_callable_p
 8)    3           1.74     0.58    1.14%      1.63     0.54    1.07%  add-zle-hook-widget
 9)    9           1.56     0.17    1.02%      1.56     0.17    1.02%  is-at-least
10)   12           1.50     0.12    0.98%      1.50     0.12    0.98%  compdef
11)    1           1.20     1.20    0.78%      1.20     1.20    0.78%  colors
12)    7           1.07     0.15    0.70%      1.07     0.15    0.70%  add-zsh-hook
13)    1           0.88     0.88    0.57%      0.88     0.88    0.57%  regexp-replace
14)    1           0.26     0.26    0.17%      0.26     0.26    0.17%  (anon) [/Users/ankit/.oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh:458]
15)    6           0.26     0.04    0.17%      0.26     0.04    0.17%  is_plugin
16)    1           0.19     0.19    0.13%      0.19     0.19    0.13%  zvm_exist_command
17)    1           0.11     0.11    0.07%      0.11     0.11    0.07%  (anon) [/usr/local/Cellar/zsh/5.9/share/zsh/functions/add-zle-hook-widget:28]
18)    3           0.09     0.03    0.06%      0.09     0.03    0.06%  is_theme
19)    2           0.08     0.04    0.05%      0.08     0.04    0.05%  env_default
20)    1           0.02     0.02    0.02%      0.02     0.02    0.02%  bashcompinit
21)    1           0.02     0.02    0.01%      0.02     0.02    0.01%  _zsh_highlight__is_function_p
22)    1           0.00     0.00    0.00%      0.00     0.00    0.00%  _zsh_highlight_bind_widgets
```

* As a reminder, whenever `.zshrc` is changed. Saving then reloading it will ensure all changes are reflected. Reload with `source .PATH/TO/.zshrc`.

* So if we run the below command:

```sh
time zsh -i -c exit && time zsh -i -c "nvm --version" exit
```

* The output will show an increased load time with `nvm`.

```TEXT
Saving session...completed.
zsh -i -c exit  0.32s user 0.30s system 92% cpu 0.670 total
Restored session: Mon Feb 17 21:34:18 CST 2025
0.40.1

Saving session...completed.
zsh -i -c "nvm --version" exit  0.73s user 0.90s system 99% cpu 1.641 total
```

* `bash` or `zsh` will read the commands left to right
* `time zsh -i -c exit` will complete before `time zsh -i -c "nvm --version" exit`.
* `0.32s user` v. `0.73s user`
* `0.30s system` v. `0.90s system`

## 3. References

* [gnu time](https://www.gnu.org/software/time/)
* [zmodload](https://zsh.sourceforge.io/Doc/Release/Shell-Builtin-Commands.html#index-zmodload)
* [zsh/zprof](https://zsh.sourceforge.io/Doc/Release/Zsh-Modules.html#The-zsh_002fzprof-Module)
