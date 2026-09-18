# Incident Log

Documented troubleshooting tickets from simulated failure scenarios in the home data center lab.

---

## Ticket #001 — Switch Port Disabled

- **Date:** 2026-08-13
- **Priority:** Medium
- **Symptom:** Laptop lost network connectivity via Ethernet; ping to gateway (192.168.10.1) timed out.
- **Initial observation:** Switch Port 3 link light was off, despite the cable being physically undisturbed.
- **Diagnosis:** Checked the switch's Port Setting page and confirmed Port 3 status was set to "Disable."
- **Root cause:** Port was administratively disabled via switch configuration.
- **Resolution:** Re-enabled Port 3 in the switch's Port Setting page.
- **Verification:** Port 3 link light returned to active state; ping to 192.168.10.1 succeeded (0% packet loss).

---

## Ticket #002 — IP Misconfiguration

- **Date:** 2026-08-13
- **Priority:** Medium
- **Symptom:** Laptop lost network connectivity via Ethernet; ping to gateway (192.168.10.1) timed out.
- **Initial observation:** No physical link issues — switch port light remained on, cable undisturbed.
- **Diagnosis:** Ran `ipconfig /all` and found the laptop's IPv4 address had been changed to 192.168.11.10, while the default gateway remained 192.168.10.1 — placing the IP and gateway in different subnets.
- **Root cause:** Static IP misconfiguration — device IP and gateway were not in the same subnet (192.168.11.0/24 vs. 192.168.10.0/24), making the gateway unreachable.
- **Resolution:** Corrected the laptop's static IPv4 address back to 192.168.10.10, matching the 192.168.10.0/24 subnet of the gateway.
- **Verification:** Ping to 192.168.10.1 succeeded with 0% packet loss.

---

## Ticket #003 — SSH Service Down

- **Date:** 2026-08-13
- **Priority:** High
- **Symptom:** Unable to establish SSH connection to server (192.168.20.1) from laptop. Error: `ssh: connect to host 192.168.20.1 port 22: Connection refused`
- **Initial observation:** Network connectivity confirmed working (ping to 192.168.20.1 succeeded); issue isolated specifically to the SSH service, not general network routing.
- **Diagnosis:** Checked service status with `systemctl status ssh` and `ssh.socket` on the server; found both `ssh.service` and `ssh.socket` had been stopped, meaning nothing was listening on port 22.
- **Root cause:** SSH service and its associated socket listener were both manually stopped.
- **Resolution:** Restarted both services via `systemctl start ssh.socket` and `systemctl start ssh.service`.
- **Verification:** Confirmed `ssh.service` status returned to "active (running)"; successfully re-established SSH connection from laptop with password authentication.

---

## Ticket #004 — Disk Space Exhaustion

- **Date:** 2026-08-14
- **Priority:** High
- **Symptom:** Server disk usage reached 95% capacity, leaving only ~5GB free; unable to create new files (simulated with `fallocate`, which failed due to insufficient space).
- **Initial observation:** Ran `df -h` and found root partition (`/dev/mapper/ubuntu--vg-ubuntu--lv`) usage had jumped from 9% to 95%.
- **Diagnosis:** Used `du -sh` on the shared folder to identify large files consuming space; found an oversized placeholder file (`bigfile.img`, 80GB) created for testing.
- **Root cause:** Oversized file(s) in `/srv/samba/shared` consuming nearly all available disk space.
- **Resolution:** Deleted the oversized files using `rm`, freeing up the reserved disk space.
- **Verification:** Ran `df -h` and confirmed root partition usage returned to baseline (~9%).
