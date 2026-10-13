# ZylexNode Fresh Bot Bundle

This bundle contains the latest available ZylexNode bot source, a systemd unit, requirements, environment template, installer, and host/KVM diagnostics.

## Install (Ubuntu 22.04/24.04; run as root)

1. Upload and extract the ZIP on the host that will run the Discord bot.
2. Enter the extracted `zylexnode-complete-bundle` directory.
3. Run:

```bash
bash scripts/preflight.sh
bash scripts/install.sh
nano /opt/zylexnode/.env
```

4. In `.env`, set at minimum `DISCORD_TOKEN` and `MAIN_ADMIN_ID`. Review every node/backend/DB setting. **Do not replace an existing production `.env` or database with the example.** The installer backs up an existing `/opt/zylexnode` directory and preserves its `.env` if present.
5. Start and inspect:

```bash
bash /opt/zylexnode/scripts/doctor.sh
bash /opt/zylexnode/scripts/kvm-doctor.sh
systemctl enable --now voxelcloud-bot
systemctl restart voxelcloud-bot
systemctl status voxelcloud-bot --no-pager -l
journalctl -u voxelcloud-bot -n 100 --no-pager -o cat
```

## KVM SSH / SSHX requirements

- SSHX is created **inside the guest OS**. The guest needs working outbound internet and the SSHX binary/runtime. Official SSHX install instructions: https://sshx.io/.
- Host-side QEMU Guest Agent support requires the libvirt virtio channel `org.qemu.guest_agent.0` **and** a running `qemu-guest-agent` service inside the guest. Host-side channel configuration alone cannot install/start software in a guest that has not booted or has no network. See https://wiki.libvirt.org/Qemu_guest_agent.html and https://www.qemu.org/docs/master/interop/qemu-ga.html.
- The bot now avoids a long IPv4 DHCP wait in the KVM SSHX flow and tries a short Guest Agent check. It cannot magically repair a dead/unbooted guest or a disconnected QGA channel. If QGA is offline and SSH port 22 is refusing connections, restore the guest boot/network/agent first using a console or rescue method. A DHCP lease alone does not prove SSH or internet works.
- Do not `virsh destroy`, `kill -9` QEMU, or edit/copy a live qcow2 disk to troubleshoot. Back up the VM after a clean shutdown. Never expose root passwords or SSH tunnel links publicly.
- SSH over Pinggy/public tunnel requires the guest to run SSH and establish an outbound tunnel. It does not mean the VM has a directly reachable public IPv4.

## Disk sizes

The admin resource picker offers larger virtual-disk choices including 256 GB, 512 GB, 1 TB (1024 GB), 2 TB (2048 GB), and 4 TB (4096 GB). These are virtual sizes, not reserved physical storage. Check real free host storage and configured allocation limits before granting large disks. Existing free-user limits remain controlled by the bot's allow-list/configuration.

## Service name

The included systemd unit is `voxelcloud-bot.service`; use `systemctl restart voxelcloud-bot`, not `systemctl restart bot`.

## Safety / verification

- Installer syntax-checks `bot.py`, creates a Python virtual environment, installs Python dependencies, and enables the service.
- Existing `/opt/zylexnode` is copied to a timestamped backup before bundle files are copied. Existing `.env` is preserved.
- The bot and scripts have been syntax-checked locally, but this bundle has **not** been integration-tested on your live host, Discord server, guest images, or provider network. Run diagnostics and test one disposable KVM guest before enabling broad user deployment.
- Rotate any passwords or tokens that have been pasted into chat or logs. Keep `.env` mode 600 and restrict Discord command permissions.
