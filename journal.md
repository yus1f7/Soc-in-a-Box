# Journal: Week 1

## 2026-10-13: Built the three VMs and took first snapshots
- Created siem-host (16 GB, 6 vCPU, 250 GB), ubuntu-web (2 GB) and win-client (4 GB, 80 GB) in VMware Workstation.
- Took `w1-clean-install` on all three before any Wazuh work.
- **Learned:** the snapshot comes first, so a failed Wazuh install can be rolled back in minutes.

## 2026-10-13: Wazuh single-node stack in Docker
- Installed Docker from the official repository, set `vm.max_map_count=262144` in `/etc/sysctl.conf`, and started the single-node stack.
- `docker compose ps` showed manager, indexer and dashboard up, version 4.14.0, ports 443, 1514-1515, 55000, 9200 open.
- **Idle memory:** dashboard 289 MiB, manager 563 MiB, indexer 1.48 GiB.
- **Learned:** the indexer is by far the heaviest container and requires the kernel memory map setting to stay up.

## 2026-10-14: Host-Only Network Routing & Browser Timeout
- **What failed:** Could not reach `https://10.10.10.10` from the host browser (`ERR_CONNECTION_TIMED_OUT`), even though ping worked and port 443 was listening.
- **What fixed it:** Checked `ipconfig` on the host and saw `VMnet1` was still on `192.168.90.1`. Configured `VMnet1` with static IP `10.10.10.1/24` in `ncpa.cpl`.
- **Learned:** Hypervisor virtual adapters on the host OS must match the guest static subnet for host-to-VM routing to work.

## 2026-10-14: Agent version alignment and pending status
- **What failed:** Endpoints initially had version mismatch / `ubuntu-web` stayed in `pending` status on the dashboard.
- **What fixed it:** Aligned agent versions to 4.14.0 so agents are never newer than the manager, and restarted `wazuh-agent` (`sudo systemctl restart wazuh-agent`).
- **Learned:** Always pin one Wazuh version across the whole lab and verify connection status in `/var/ossec/logs/ossec.log`.

## 2026-10-14: NAT adapter left on win-client
- **What failed:** win-client still had `Network Adapter 2: NAT`, so it still had a route to the internet. This breaks the lab isolation rule (S2).
- **What fixed it:** Removed the NAT adapter from both endpoints after installing Apache, Sysmon, and Wazuh agents, then took the `w1-done` snapshots.
- **Learned:** The lab must be isolated; always verify VM hardware settings after initial package installation.

## 2026-10-14: Five known events (V1)
- Generated: failed SSH login (rule 5710), sudo (5402), new user `testuser` (5902), `cmd.exe` through Sysmon (61603), and a failed Windows logon (event 4625, rule 60122).
- All five were found in the dashboard (Discover, last 24 h).
- **Learned:** Sysmon shows the full command line and the parent process (PowerShell started cmd.exe), which the standard Security log does not provide.

## 2026-10-14: Persistence test (W4)
- **What I did:** First, stopped and started the Wazuh stack on `siem-host` using `docker compose stop` and `docker compose start`. Second, performed a full VM reboot of `siem-host` (`sudo reboot`).
- **What I observed:** All three containers (`wazuh.manager`, `wazuh.indexer`, `wazuh.dashboard`) came back up automatically after the reboot thanks to the `restart: always` policy and persistent Docker volumes. Both `ubuntu-web` (001) and `win-client` (002) reconnected automatically as `Active`, and all previously generated alerts remained intact in the dashboard.
