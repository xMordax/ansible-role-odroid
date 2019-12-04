# ansible-role-odroid

Slightly modified [hannseman/ansible-raspbian](https://github.com/hannseman/ansible-raspbian) roles to be compatible
with a Ordoird(Ubuntu 18.04) device. It should also work with any other Debian based system.

### It will:

 * Install specified system packages.
 * Configure hostname.
 * Configure locale.
 * Mount tmpfs on write-intensive directories to increase the lifespan of SD-card. 
 * Change the password on default user.
 * Set the default editor.
 * Setup a secure SSH configuration.
 * Configure UFW.
 * Configure Postfix to send email through an SMTP relay.
 * Enable unattended-upgrades.
 * Install Fail2ban.
 * Configure Logwatch to send weekly reports.

### It will not:

 * Update system packages.
 * Run `apt-get update`. Please do this in a pre_task. See [Example Playbook](#example-playbook).
 * Install security patches but unattended-upgrades should take care of that. 

## Setup
* Install sshpass by running `sudo apt-get install sshpass`.
* Flash SD-card with.
* Run playbook.

## Inventory

[sshpass](https://linux.die.net/man/1/sshpass) is required to make the first Ansible run
with the default password `odroid` under root user. Password authentication over SSH will then be disabled in
preference of public key authentication with keys specified in `ssh_public_keys`. 
Your inventory should contain the following:

```ini
[all:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=odroid
```

## Variables

```yaml
system_hostname: "odroid"
system_ssh_user_password: "odroid"
system_ssh_user_salt: "salt"
system_locale: "en_US.UTF-8"
system_timezone: "Europe/Stockholm"
system_tmpfs_mounts:
  - { src: "/run", size: "10%", options: "nodev,noexec,nosuid" }
  - { src: "/tmp", size: "10%", options: "nodev,nosuid" }
  - { src: "/var/log", size: "10%", options: "nodev,noexec,nosuid" }

system_packages: []
system_default_editor_path: "/usr/bin/vi"
logwatch_tmp_dir: /var/cache/logwatch
logwatch_mailto: "root"
logwatch_detail: "Low"
logwatch_interval: "weekly"

postfix_hostname: "{{ ansible_hostname }}"
postfix_mailname: "{{ ansible_hostname }}"
postfix_mydestination:
  - "{{ postfix_hostname }}"
  - localdomain
  - localhost
  - localhost.localdomain
postfix_relayhost: smtp.gmail.com
postfix_relayhost_port: 587
postfix_sasl_user:
postfix_sasl_password:
postfix_smtp_tls_cafile: /etc/ssl/certs/ca-certificates.crt

ssh_sshd_config: "/etc/ssh/sshd_config"
ssh_public_keys: []
ssh_banner:

ufw_rules:
  - {rule: "allow", port: "22", proto: "tcp"}
ufw_allow_igmp: false

unattended_upgrades_email_address: root

bash_aliases: |
  ## get rid of command not found ##
  alias cd..='cd ..'

  ## a quick way to get out of current directory ##
  alias ..='cd ..'
  alias ...='cd ../../../'
  alias ....='cd ../../../../'
  alias .....='cd ../../../../'
  alias .4='cd ../../../../'
  alias .5='cd ../../../../..'

  ## Colorize the ls output ##
  alias ls='ls --color=auto'

  ## Use a long listing format ##
  alias ll='ls -laF'
  alias lla='ls -laF'

  ## Show hidden files ##
  alias l.='ls -d .* --color=auto'

  alias grep='grep --color=auto'
  alias egrep='egrep --color=auto'
  alias fgrep='fgrep --color=auto'

  # handy short cuts #
  alias h='history'
  alias j='jobs -l'

  alias vi=vim
  alias svi='sudo vi'
  alias vis='vim "+set si"'
  alias edit='vim'

  # show open ports
  alias ports='netstat -tulanp'

  # become root #
  alias root='sudo -i'
  alias su='sudo -i'


  # reboot / halt / poweroff
  alias reboot='sudo /sbin/reboot'
  alias poweroff='sudo /sbin/poweroff'
  alias halt='sudo /sbin/halt'
  alias shutdown='sudo /sbin/shutdown'

  ## pass options to free ##
  alias meminfo='free -m -l -t'

  ## get top process eating memory
  alias psmem='ps auxf | sort -nr -k 4'
  alias psmem10='ps auxf | sort -nr -k 4 | head -10'

  ## get top process eating cpu ##
  alias pscpu='ps auxf | sort -nr -k 3'
  alias pscpu10='ps auxf | sort -nr -k 3 | head -10'

  ## Get server cpu info ##
  alias cpuinfo='lscpu'

  ## set some other defaults ##
  alias df='df -H'
  alias du='du -ch'


  # top is atop, just like vi is vim
  alias top='htop'
```

## Example Playbook
```yaml
- hosts: servers
  become: true
  pre_tasks:
    - name: update apt cache
      apt:
        cache_valid_time: 600
  roles:
    - role: xmordax.odroid
  vars:
    system_packages:
      - apt-transport-https
      - vim
    system_default_editor_path: "/usr/bin/vim.basic"
    system_ssh_user_password: hunter2
    system_ssh_user_salt: pepper
    postfix_sasl_user: root@example.com
    postfix_sasl_password: hunter2

    ssh_public_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJXTGInmtpoG9rYmT/3DpL+0o/sH2shys+NwJLo8NnCj
```