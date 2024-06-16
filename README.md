# rust-stack
### Steps
- Login
```console
ssh linuxuser@host:22
```
- Update server and change password
```console
sudo dnf install epel-release -y
sudo dnf update -y
sudo passwd linuxuser
sudo passwd root
```
<!-- TODO -->
- Change SSH port
```console
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo vi /etc/ssh/sshd_config
```
```diff
- # Port 22
+ Port 12345

:wq
```
```console
sudo semanage port -a -t ssh_port_t -p tcp 12345
sudo firewall-cmd --add-port=12345/tcp --permanent
sudo firewall-cmd --remove-service=ssh --permanent
sudo systemctl restart sshd
sudo systemctl restart firewall-cmd
# Confirm SSH is still working in new terminal tab
sudo netstat -tlpn| grep ssh
```
- Install fail2ban
```console
sudo dnf install fail2ban -y
sudo systemctl enable --now fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vi /etc/fail2ban/jail.local
```
```diff
- # bantime = 10m
+ bantime = 1h
- # findtime = 10m
+ findtime = 1h

[sshd]
# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
+ enabled = true
+ filter  = sshd
+ maxretry= 3
+ bantime = 3600

:wq
```
```console
sudo mv /etc/fail2ban/jail.d/00-firewalld.conf /etc/fail2ban/jail.d/00-firewalld.local
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
```
- Install Caddy
<!-- TODO -->


### Useful commands
- `sudo netstat -tlpn| grep ssh`
```
tcp        0      0 0.0.0.0:22           0.0.0.0:*               LISTEN      4933/sshd: /usr/sbi
tcp6       0      0 :::22                :::*                    LISTEN      4933/sshd: /usr/sbi
```
- `ssh-keygen -R host` Generates new SSH key for a host, useful for changed host identity after server reinstall
