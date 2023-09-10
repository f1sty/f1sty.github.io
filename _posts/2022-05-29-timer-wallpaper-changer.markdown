---
layout: post
title:  "Changing wallpaper in dwm periodically using systemd.timer"
date:   2022-05-29 11:24:40 +0300
categories: archlinux linux systemd dwm wallpaper rice feh
---
I've heard about this fancy wallpaper changers, that sits in backgound and set a random wallpaper
periodically, so I've decided to get myself one.

### UPDATE:
Since the timer won't start automatically if you not using GUI-login, you can put `systemctl
start --user wallpaper.timer` into your `.xinitrc`.

### Installing needed programs
First of all, lets install something, that could be used to setup wallpaper. I'm using `feh`, it's
small and fast image viewer and it can set backgroud also:
```bash
$ pacman -S feh
```

### Setting up programs
Once you've installed `feh`, you should run it like this:
```bash
$ feh --bg-scale --randomize /path/to/wallpapers/*
```

Options explanation:
- `--bg-scale` option is:
> Fit the file into the background without repeating it, cutting off stuff or using borders.  But
> the aspect ratio is not preserved either.
- `--randomize` is basically choosing random file from the list, as you can imagine.

After you run it once, it will create script at your home directory called `.fehbg` (unless you
specify `--no-fehbg` option;)), and you should use this one in your service file, that we are
about to create.

### Creating systemd service
Now, we should create a systemd service file, that will run `.fehbg` script created earlier.
Choose a name for your service, I've got very creative `wallpaper.service`.Then run this in
your shell:
```bash
$ systemctl edit --user --full --force wallpaper.service
```

It will open your default editor with an empty service unit file.

Options explanation:
- `--user` means we creating user-level systemd service, that will be run only for this user on
  user's login. It's usually stored in `$HOME/.config/systemd/user` directory.
- `--full` will create or copy (if there is existing one) a full unit file, instead of overriding
  existing unit file.
- `--force` ensures, that if there is no such existing unit file, it will be created from scratch.

Than put this into newly created file, save and exit:
```
[Unit]
Description=Setup random wallpaper

[Service]
Type=oneshot
Environment="DISPLAY=:0"
ExecStart=/path/to/.fehbg
```

This unit file will run your `.fehbg` script, you should put path to your `.fehbg` script after
`ExecStart=`. `Type=oneshot` means the service manager will consider the unit up after the main
process exits. If you want more details, read `man systemd.service` and `man systemd.unit`.
I'm also setting up DISPLAY environment variable for the script, just in case:).

### Creating systemd timer
After we've created wallpaper changer service, it's time to create systemd timer. For simplicity,
name it same name, as a service name, beacuse by default, a service by the same name as the timer
(except for the suffix) is activated:
```bash
$ systemctl edit --user --full --force wallpaper.timer
```

We've already familiar with this options, so put this into new timer unit file:
```
[Unit]
Description=Update wallpaper periodically

[Timer]
OnUnitActiveSec=10m

[Install]
WantedBy=timers.target
```

Option `OnUnitActiveSec=` defines a timer relative to when the unit the timer unit is activating
was last activated. In our case, this means relative to the last time `wallpaper.service` was
activated. The time format in this option is quite flexible, you can read more about it in
`man systemd.timer`, but in our case we just set it to `10m`, which makes timer run our service
unit file every 10 minutes.

### Wrap up
Well, that's it. Now your background will always be fresh. I encourage you to read man pages on
`systemd`, `systemd.unit`, `systemd.service` and `systemd.timer` to get a better grasp on the
details.

Have a nice and productive day!
