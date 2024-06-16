# rust-stack
### Prerequisites
An internet connection, almost any machine and a fresh installation of Rocky 9
### Steps
- Login
```console
ssh linuxuser@133.37.133.37:22
```
- Update server and change password
```console
sudo dnf install epel-release -y
sudo dnf upgrade --refresh -y
sudo dnf update --refresh -y
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
port    = ssh

:wq
```
```console
sudo mv /etc/fail2ban/jail.d/00-firewalld.conf /etc/fail2ban/jail.d/00-firewalld.local
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
sudo cat /var/log/fail2ban.log
# Confirm fail2ban is working (new terminal tab, dont get yourself banned)
sudo cat /var/log/fail2ban.log
```
- Compile [sqlite3](https://www.sqlite.org/index.html) from [source](https://github.com/sqlite/sqlite) (RHEL is living in 2021 iirc)
```console
cd ~
wget https://www.sqlite.org/src/tarball/sqlite.tar.gz
tar xzf sqlite.tar.gz
mkdir build
cd build
~/sqlite/configure
make verify-source
make
make sqlite3.c # make: 'sqlite3.c' is up to date.
sudo mv sqlite3 /usr/bin/sqlite3
sqlite3 --version # 3.47.0 2024-06-14 23:13:54 13242289c5d412b706f50fc7e1553032ea3a52d41a3e34e155432adaf0551481 (64-bit)
cd ~
rm -rf build sqlite sqlite.tar.gz
```
- Install [Caddy](https://linuxiac.com/installing-caddy-php-on-rocky-linux-9-almalinux-9/)
```console
sudo dnf install 'dnf-command(copr)' -y
sudo dnf copr enable @caddy/caddy -y
sudo dnf install caddy -y # caddy-2.8.1-1.el9.x86_64
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
# in case that didnt work: `sudo systemctl restart firewalld`
```
A properly configured firewall will look something [like this](https://docs.rockylinux.org/de/guides/web/caddy/)
```yaml
$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp1s0
  sources:
  services: cockpit dhcpv6-client http https
  ports: 12345/tcp # Our new SSH port
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

### Useful commands
- `sudo netstat -tlpn| grep ssh`
```
tcp        0      0 0.0.0.0:22           0.0.0.0:*               LISTEN      4933/sshd: /usr/sbi
tcp6       0      0 :::22                :::*                    LISTEN      4933/sshd: /usr/sbi
```
- `ssh-keygen -R host` Generates new SSH key for a host, useful for changed host identity after server reinstall
- `curl -4 icanhazip.com` 133.37.133.37
