# systemd Cheatsheet — Commands That Actually Work

One page. Thirty entries: `systemctl` service management, `journalctl` logs, unit files, timers, and system state. Each entry tells you **what it does**, **a command you can copy-paste**, and **tags**. No made-up flags — every command and option below was checked against systemd 255 (`systemctl --help` / `journalctl --help` / `systemd-analyze --help` output), and the two example units were validated for real with `systemd-analyze verify`; the calendar expressions were run through `systemd-analyze calendar`. Most management commands need `sudo`. One precondition: `systemctl` only works on a system actually booted with systemd — inside containers and other non-systemd environments it fails with "System has not been booted with systemd as init system", so check your environment first (`ps -p 1 -o comm=` should print `systemd`).

> 🌍 Read this project in your language: [中文](README.md) · [Español](README.es.md) · [Português](README.pt.md)

Docs: https://www.freedesktop.org/software/systemd/man/latest/

---

## 🎛️ Service management — systemctl

### `systemctl status unit` — see whether a unit is running, failed, or dead
```bash
sudo systemctl status nginx            # state, main PID, memory, recent log lines
sudo systemctl status nginx --no-pager # don't pipe into a pager — for scripts and docs
# status opens in a pager by default: press q to quit
```
🏷️ `#services` `#status` `#must-know`
### `systemctl start / stop unit` — start or stop a unit right now
```bash
sudo systemctl start nginx   # start it (and pull in its dependencies)
sudo systemctl stop nginx    # stop it — nothing about boot behaviour changes
```
🏷️ `#services` `#basics`
### `systemctl restart` vs `reload` — restart bounces it, reload re-reads config
```bash
sudo systemctl restart nginx          # full stop + start; drops connections
sudo systemctl reload nginx           # re-read config WITHOUT restarting, if the unit supports it
sudo systemctl reload-or-restart nginx # reload if supported, otherwise restart — the safe default
# reload on a unit with no reload support fails with
# "Reload request not supported for this unit" — that's when reload-or-restart earns its keep
```
🏷️ `#services` `#reload` `#must-know`
### `systemctl enable / disable unit` — control boot-time start, and ONLY that
```bash
sudo systemctl enable nginx    # create the boot symlink — does NOT start the service now
sudo systemctl disable nginx   # remove the symlink — does NOT stop a running service
# the classic trap: enable ≠ start. A freshly enabled service only comes up at next boot.
```
🏷️ `#services` `#boot` `#gotcha`
### `enable --now / disable --now` — set boot behaviour AND start/stop in one step
```bash
sudo systemctl enable --now nginx    # enable at boot + start right now
sudo systemctl disable --now nginx   # disable at boot + stop right now
```
🏷️ `#services` `#boot` `#must-know`
### `is-active / is-enabled / is-failed` — ask yes/no questions, read the exit code
```bash
systemctl is-active nginx    # prints active / inactive / failed / activating / ...
systemctl is-enabled nginx   # prints enabled / disabled / static / masked / ...
systemctl is-failed nginx    # prints "failed" + exit 0 ONLY if the unit really is failed;
                             # otherwise it prints the real state (active / inactive) + a non-zero exit
# all three are built for scripts: exit code 0 = the answer is "yes"
# (is-active: unit is active; is-enabled: unit is enabled; is-failed: unit is failed),
# non-zero = no / unit not found
systemctl is-active --quiet nginx && echo running   # --quiet: no output, just the exit code
```
🏷️ `#services` `#scripting` `#exit-codes`
### `systemctl mask / unmask unit` — ban a unit from starting at all
```bash
sudo systemctl mask nginx      # symlink the unit to /dev/null
sudo systemctl unmask nginx    # undo it
```
⚠️ A masked unit cannot be started **at all** — not manually, not as a dependency of another unit. `start` fails with "Unit is masked". Use it to permanently silence a unit you never want back, not as a stronger `disable`.
🏷️ `#services` `#mask` `#danger`
### `systemctl daemon-reload` — re-read unit files after editing them
```bash
sudo systemctl daemon-reload   # run this after ANY edit under /etc/systemd/system or /usr/lib/systemd/system
sudo systemctl daemon-reexec   # re-execute the manager itself (rare — mostly after a systemd upgrade)
# skip daemon-reload and systemd keeps using the old in-memory copy, silently
```
🏷️ `#services` `#units` `#must-know`

---

## 🔍 Logs — journalctl

### `journalctl -u unit` — show one unit's logs
```bash
journalctl -u nginx                    # this unit only, oldest first, in a pager (q to quit)
journalctl -u nginx -e                 # -e: jump straight to the end (newest entries)
journalctl -u nginx --no-pager -n 50   # last 50 lines, no pager — safe to copy or script
# related: journalctl -k shows kernel messages (dmesg) from the current boot
```
🏷️ `#logs` `#journal` `#must-know`
### `journalctl -f` — follow logs live
```bash
journalctl -f                          # tail the whole journal, Ctrl-C to stop
journalctl -f -u nginx -u php-fpm      # -u stacks: follow several units in one stream
```
🏷️ `#logs` `#follow`
### `journalctl -b` — logs per boot
```bash
journalctl -b         # this boot only
journalctl -b -1      # the PREVIOUS boot — where you look after a crash or a hang
journalctl --list-boots   # list recorded boots with IDs and time ranges, then pick with -b <id>
```
🏷️ `#logs` `#boot` `#troubleshooting`
### `journalctl --since / --until` — cut a time window
```bash
journalctl --since "1 hour ago"                      # relative times just work
journalctl --since "2026-10-20 09:00" --until "2026-10-20 10:00"  # absolute window
journalctl -u nginx --since today                    # today / yesterday are valid too
```
🏷️ `#logs` `#time`
### `journalctl -p err` — filter by priority
```bash
journalctl -p err          # errors and worse (this boot)
journalctl -p warning -b -1  # warnings and worse, previous boot
# levels, worst to noisiest: 0 emerg, 1 alert, 2 crit, 3 err,
# 4 warning, 5 notice, 6 info, 7 debug
# -p err means "err and above" (0-3); a range like -p info..err also works
```
🏷️ `#logs` `#priority` `#troubleshooting`
### `journalctl --disk-usage / --vacuum-time` — see and shrink the journal
```bash
journalctl --disk-usage              # how much disk archived + active journals take
sudo journalctl --vacuum-time=2weeks # delete archived journals older than 2 weeks
sudo journalctl --vacuum-size=500M   # or shrink the archive to a total size instead
```
⚠️ Vacuum **deletes** old log files permanently — there is no undo. Run `--disk-usage` first, and don't vacuum right before investigating an incident: the history you need is exactly what gets deleted.
🏷️ `#logs` `#cleanup` `#danger`

---

## 📄 Unit files

### `systemctl cat unit` — print the unit as systemd actually sees it
```bash
systemctl cat nginx   # base file + every drop-in merged, each piece labelled with its file path
# this is the ground truth — what you edited is not always what is running
```
🏷️ `#units` `#drop-ins`
### Unit locations & precedence — where unit files live
```bash
systemctl list-unit-files            # every installed unit file + its enable state
systemctl list-unit-files --type=service   # narrow it to services
# load order (first hit wins):
#   /etc/systemd/system        ← local admin units & drop-ins, highest priority
#   /run/systemd/system        ← runtime units, gone at reboot
#   /usr/lib/systemd/system    ← vendor/package units (older docs say /lib/systemd/system — same place on merged-/usr systems)
# never edit the vendor copy in /usr/lib — your change is overwritten on package upgrade; override in /etc instead
```
🏷️ `#units` `#paths` `#must-know`
### Anatomy of a `.service` — a complete, verified example
```ini
[Unit]
Description=MyApp demo service     # human-readable name, shown by status
After=network.target               # start AFTER the network is up (ordering only)
Wants=network.target               # pull network.target in with us (weak requirement)

[Service]
Type=simple                        # ExecStart's main process IS the service
ExecStart=/usr/bin/sleep infinity  # placeholder — put your real program path here (must exist; verify checks)
Restart=on-failure                 # restart on non-zero exit / signal, not on clean exit
User=www-data                      # run unprivileged, not as root (use your app's own user)

[Install]
WantedBy=multi-user.target         # "enable" hooks this unit into the normal multi-user boot
```
```bash
systemd-analyze verify ./myapp.service   # this exact file passes verify with no warnings
```
🏷️ `#units` `#service-file` `#must-know`
### `systemctl edit unit` — override with a drop-in, don't touch the vendor file
```bash
sudo systemctl edit nginx            # opens an editor for /etc/systemd/system/nginx.service.d/override.conf
sudo systemctl edit --full nginx     # --full: edit a full copy of the unit itself instead of a drop-in
# drop-ins win over the vendor file and survive package upgrades;
# after saving, edit runs daemon-reload for you — a manual unit-file edit does not
```
🏷️ `#units` `#drop-ins` `#editing`
### `systemctl show unit -p` — read one property exactly
```bash
systemctl show nginx -p MainPID          # main process PID (0 when not running)
systemctl show nginx -p MemoryCurrent    # current memory use in bytes ([not set] if unavailable)
systemctl show nginx -p Restart,User     # several properties at once
systemctl show nginx                     # no -p: dump every property
```
🏷️ `#units` `#inspect` `#scripting`
### `systemctl list-dependencies unit` — see the dependency tree
```bash
systemctl list-dependencies nginx        # what this unit needs, recursively, as a tree
systemctl list-dependencies --reverse nginx  # the other way: who needs THIS unit
systemctl list-dependencies multi-user.target  # everything a normal boot pulls in
```
🏷️ `#units` `#dependencies`
### `systemctl --failed` + `reset-failed` — find and clear failures
```bash
systemctl --failed                       # list units in failed state (same as list-units --state=failed)
systemctl status <the-unit>              # then read WHY it failed here, and in journalctl -u
sudo systemctl reset-failed nginx        # clear the failed state (all units, if you name none)
# reset-failed only clears the flag — fix the cause first, or it just fails again
```
🏷️ `#units` `#troubleshooting` `#must-know`

---

## ⏰ Timers — systemd cron replacement

### `systemctl list-timers --all` — every timer, next run included
```bash
systemctl list-timers --all   # NEXT / LEFT / LAST / PASSED + which service each timer activates
# without --all you only see timers that are loaded and active — --all also shows inactive ones
```
🏷️ `#timers` `#list` `#must-know`
### Anatomy of a `.timer` — a daily job, with its service
```ini
# backup.timer
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=daily        # calendar trigger: daily = *-*-* 00:00:00; hourly, weekly, "*-*-* 03:00" also work
Persistent=true         # missed while powered off? run once at next boot instead of skipping
Unit=backup.service     # what to start — defaults to the same-named .service, so this line is optional here

[Install]
WantedBy=timers.target  # timers hook into timers.target, not multi-user.target
```
```ini
# backup.service — the job itself (kept minimal: a timer starts it, nothing enables it)
[Unit]
Description=Nightly backup job

[Service]
Type=oneshot            # run once and exit — the right type for timer jobs
ExecStart=/bin/true     # placeholder — put your real script/command here (must exist; verify checks)
```
```bash
systemd-analyze verify ./backup.service ./backup.timer   # this exact pair passes verify
```
🏷️ `#timers` `#timer-file` `#must-know`
### `systemd-analyze calendar` — test a calendar expression before you ship it
```bash
systemd-analyze calendar "Mon *-*-* 03:00"
#   Original form: Mon *-*-* 03:00
#   Normalized form: Mon *-*-* 03:00:00
#       Next elapse: Mon 2026-10-26 03:00:00 UTC   (example output — the date moves with the day you run it)
systemd-analyze calendar daily   # → Normalized form: *-*-* 00:00:00, plus its next elapse
```
🏷️ `#timers` `#calendar` `#verify`
### `systemctl enable --now xxx.timer` — enable the TIMER, not the service
```bash
sudo systemctl enable --now backup.timer   # start the countdown + enable at boot
systemctl list-timers --all                # confirm it shows a NEXT time
# the classic trap: enabling backup.service does nothing useful here —
# the timer is the thing that must be enabled and running
```
🏷️ `#timers` `#boot` `#gotcha`
### Relative triggers — `OnBootSec` / `OnUnitActiveSec` alongside `OnCalendar`
```ini
[Timer]
OnBootSec=5min            # fire 5 minutes after boot
OnUnitActiveSec=1h        # then again every hour after the last activation
# OnCalendar = wall clock ("every Monday 03:00"); OnBootSec / OnUnitActiveSec /
# OnUnitInactiveSec = stopwatch timers relative to boot or to the unit's own runs.
# They combine freely in one .timer — e.g. OnBootSec for a first run, OnCalendar after that.
```
🏷️ `#timers` `#relative`

---

## 🧰 System state

### `systemd-analyze` / `blame` / `critical-chain` — where did boot time go?
```bash
systemd-analyze                  # total startup time (firmware / loader / kernel / userspace)
systemd-analyze blame            # every unit, slowest first — start optimising at the top
systemd-analyze critical-chain   # the chain that actually gated boot (blame alone misleads:
                                 # a slow unit off the critical path costs you nothing)
```
🏷️ `#boot` `#performance` `#analyze`
### `get-default / set-default` — choose the boot target
```bash
systemctl get-default                        # usually graphical.target or multi-user.target
sudo systemctl set-default multi-user.target # boot to console, no GUI, from now on
```
⚠️ `set-default` is persistent. Point a desktop at `multi-user.target` and the next boot has no graphical login — recover by booting with `systemd.unit=graphical.target` on the kernel command line (or from a rescue boot) and setting it back. On a headless server, `multi-user.target` is the right default; just set it on purpose, not by accident.
🏷️ `#boot` `#targets` `#danger`
### `reboot / poweroff` and `isolate` — stopping the machine, stopping everything else
```bash
sudo systemctl reboot      # reboot (same family: systemctl poweroff, systemctl halt)
sudo systemctl isolate rescue.target   # stop almost everything, land in rescue mode — no reboot
# isolate starts ONE target and stops all units not part of it; the old runlevel
# commands (telinit / systemctl isolate runlevel3.target) map onto these same targets
```
⚠️ On a remote machine, think before `poweroff`: unlike `reboot` there is no coming back without physical access or working out-of-band management (IPMI / cloud console). Finish your SSH sessions' work first — `isolate` and `poweroff` kill them without asking.
🏷️ `#power` `#targets` `#danger`
### `hostnamectl / timedatectl / localectl` — the *-ctl trio for machine identity
```bash
hostnamectl status                                   # hostname, chassis, machine ID, OS
sudo timedatectl set-timezone America/Los_Angeles    # set the timezone (timedatectl status shows it; list-timezones lists all)
localectl status                                     # system locale + keyboard layout
```
🏷️ `#system` `#hostname` `#timezone` `#locale`

---

## Contributing

Found a wrong command, an outdated flag, or a bad translation? Pull requests are welcome — keep entries to the format above (one-liner / code block / tags) and **only real, verified commands; when in doubt, leave it out**.

## License

MIT © 2026 Guangyi Zhao. See [LICENSE](LICENSE).
