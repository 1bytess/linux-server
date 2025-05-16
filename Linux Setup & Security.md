# **Linux Setup & Security**

## First Boot Setup

1. Make sure Linux is Updated

```bash
sudo apt update && sudo apt upgrade -y
```

1. Installing NVIDA CUDA Driver Version 12.4

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
```

```bash
sudo apt-get update
sudo apt-get install -y cuda-drivers-550
```

1. Installing NVIDIA cuDNN package

```bash
sudo apt-get update
sudo apt-get -y install cudnn9-cuda-12
```

1. Secure SSH

```bash
sudo nano /etc/ssh/sshd_config
```

```
# Undashed these lines.. and set to 'no'

Port <port>
PermitRootLogin no
PasswordAuthentication no
```

## Security and Hardening

1. **Enable Automatic Updates**
   1. Install the required package:

      ```
      sudo apt update
      sudo apt install unattended-upgrades
      ```
   2. Enable automatic updates:

      ```
      sudo dpkg-reconfigure --priority=low unattended-upgrades
      ```
   3. Customize the configuration: Edit `/etc/apt/apt.conf.d/50unattended-upgrades`:

      ```
      sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
      ```

      Ensure these lines are uncommented:

      ```
      "${distro_id}:${distro_codename}-security";
      "${distro_id}:${distro_codename}-updates";
      ```
   4. Test unattended upgrades:

      ```
      sudo unattended-upgrade --dry-run
      ```
2. **Install Fail2Ban**
   1. Install the packages:

      ```bash
      sudo apt update
      sudo apt install fail2ban
      ```
   2. Edit the default config file:

      ```
      sudo nano /etc/fail2ban/jail.local
      ```
   3. Add or update these settings:

      ```
      [DEFAULT]
      bantime = 10m
      findtime = 10m
      maxretry = 3
      backend = systemd
      
      [sshd]
      enabled = true
      port = <your_custom_ssh_port>
      filter = sshd
      logpath = /var/log/auth.log
      maxretry = 3
      ```
   4. Restart Fail2Ban:

      ```
      sudo systemctl restart fail2ban
      ```
   5. Check Fail2Ban status:

      ```
      sudo fail2ban-client status
      ```
3. **Set Up Backups Using Cron**

   Automate backups to a secure location. For example, to back up `/mnt/drive/{jupyter,appdata}` to `/mnt/drive/backups`:
   1. Create a backup script:

      ```
      sudo nano /usr/local/bin/backup_script.sh
      ```
   2. Add this to the script:

      ```
      #!/bin/bash
      TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")
      SOURCE="/mnt/drive/{jupyter,appdata}"
      DESTINATION="/mnt/drive/backups"
      tar -czf $DESTINATION/backup_$TIMESTAMP.tar.gz $SOURCE
      ```
   3. Make it executable:

      ```
      sudo chmod +x /usr/local/bin/backup_script.sh
      ```
   4. Schedule it in cron: Open the cron editor:

      ```
      crontab -e
      ```

      Add this line to back up daily at 2 AM:

      ```
      0 2 * * * /usr/local/bin/backup_script.sh
      ```
4. **Additional Security**
   1. **Disable unused services:** Check running services and disable unnecessary ones:

      ```
      sudo systemctl list-units --type=service
      sudo systemctl disable <service_name>
      ```
   2. **Install and configure AppArmor:** Ensure AppArmor is enabled and profiles are enforced:

      ```
      sudo apt install apparmor apparmor-utils
      sudo aa-enforce /etc/apparmor.d/*
      ```
   3. **Monitor system logs with Logwatch:** Install and configure Logwatch to get daily reports:

      ```
      sudo apt install logwatch
      sudo logwatch --detail high --mailto your-email@example.com --output mail
      ```
   4. **Use rootkit detection tools:** Install and run a tool like RKHunter:

      ```
      sudo apt install rkhunter
      sudo rkhunter --check
      ```
5. **Verify Security**

   Regularly scan your server using tools like **Lynis**:

   ```
   sudo apt install lynis
   sudo lynis audit system
   ```