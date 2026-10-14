# Journal: Week 1

> Draft written from my screenshots and notes. Edit it in your own words before submitting.

## 2026-10-13: Built the three VMs and took first snapshots
- Created siem-host (16 GB, 6 vCPU, 250 GB), ubuntu-web (2 GB) and win-client (4 GB, 80 GB) in VMware Workstation.
- Took `w1-clean-install` on all three before any Wazuh work.
- **Learned:** the snapshot comes first, so a failed Wazuh install can be rolled back in minutes.

## 2026-10-13: Wazuh single-node stack in Docker
- Installed Docker from the official repository and started the single-node stack.
- `docker compose ps` showed manager, indexer and dashboard up, version 4.14.0, ports 443, 1514-1515, 55000, 9200 open.
- **Idle memory:** dashboard 289 MiB, manager 563 MiB, indexer 1.48 GiB.
- **Learned:** the indexer is by far the heaviest container.

## 2026-10-14: Agent version was newer than the manager
- **What failed:** Endpoints showed `ubuntu-web` on agent v4.14.8 while the manager runs 4.14.0. An agent must not be newer than its manager.
- **What fixed it:** aligned the versions so that the agents are not newer than the manager.
- **Learned:** pin one Wazuh version everywhere. The "Deploy new agent" wizard can offer the newest package.

## 2026-10-14: NAT adapter left on win-client
- **What failed:** win-client still had `Network Adapter 2: NAT`, so it still had a route to the internet. This breaks the lab isolation rule.
- **What fixed it:** removed the NAT adapter from the endpoints and re-took `w1-done`.
- **Learned:** the lab must be isolated, so check VM settings after installing software.

## 2026-10-14: Five known events (V1)
- Generated: failed SSH login (rule 5710), sudo (5402), new user `testuser` (5902), `cmd.exe` through Sysmon, and a failed Windows logon (event 4625, logon type 2, status 0xc000006d / 0xc000006a).
- All five were found in the dashboard (Discover, last 24 h).
- **Learned:** Sysmon shows the full command line and the parent process (PowerShell started cmd.exe), which the Security log does not.

## Persistence test (W4)
> Fill in: what you did (stop/start, reboot) and what you observed, and whether agents and alerts survived.
