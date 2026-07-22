# install-kstars-indi
**Ansible scripts to install KStars and INDI Library**

Distribution required:
kubuntu-26.04-desktop-amd64

Everything's already set up for a local run — inventory.ini has localhost with ansible_connection=local. Here's the sequence:

1. Install Ansible on the machine itself
`bash
sudo apt update
sudo apt install -y ansible-core python3-apt`

python3-apt matters more than it looks: Ansible's apt module needs it on the target, and since the target is this machine, a missing python3-apt fails the very first task in a chicken-and-egg way. It's usually preinstalled on Kubuntu/Debian desktops, but installing it explicitly costs nothing.

(ansible instead of ansible-core also works — the playbook only uses ansible.builtin modules, so the full collection bundle is unnecessary.)

2. Point it at your account

inventory.ini line 3 has a placeholder username. Either edit it, or just override on the command line — simplest is to let the shell fill it in:

`bash
cd astro-ansible
ansible-playbook -i inventory.ini install-kstars-indi.yml \
  -e astro_user=$USER --ask-become-pass`

--ask-become-pass (short: -K) prompts once for your sudo password. Skip it if you have passwordless sudo.

**A gotcha worth knowing**

You can't shortcut the inventory file with the usual -i localhost, -c local trick. The play targets hosts: astro, and an inline -i localhost, puts the host in all instead — Ansible will report "no hosts matched" and exit cleanly, which reads like success if you're not watching. Use the inventory file, or change the hosts: line to localhost.

**Before committing to a full run**
`bash
ansible-playbook -i inventory.ini install-kstars-indi.yml --syntax-check
ansible-playbook -i inventory.ini install-kstars-indi.yml --list-tasks`

Avoid --check here. Dry-run mode skips command tasks, so the tasks that resolve git tags and probe package availability never register their variables, and everything downstream fails on undefined vars. The failures would be artifacts of check mode, not real problems.

**If you're on Debian with the source path**

The build is long and mostly silent. Run it detached so a dropped terminal doesn't kill it:

`bash
tmux new -s astro
ansible-playbook -i inventory.ini install-kstars-indi.yml -e astro_user=$USER -K -v`

The -v gives you per-task output so you can tell it's alive during the indi-3rdparty stage, which is by far the longest. If RAM is tight, add -e build_jobs=2 — the ZWO and QHY SDKs are the ones that OOM.

**Afterwards**

Log out and back in before plugging anything in. The dialout group membership won't apply to your current session, and mount serial connections will fail with a permissions error that looks like a driver problem but isn't.

Log out and back in before plugging anything in. The dialout group membership won't apply to your current session, and mount serial connections will fail with a permissions error that looks like a driver problem but isn't.
