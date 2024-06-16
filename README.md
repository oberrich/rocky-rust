# rust-stack
### Useful commands
- `ssh-keygen -R host`: Generates new SSH key for a host, useful for changed host identity after server reinstall.

### Steps
- Update server and change password
```bash
ssh linuxuser@host:22
sudo dnf install epel-release -y
sudo dnf update -y
sudo passwd linuxuser
sudo passwd root
```
<!-- TODO -->
- Change SSH port
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo vi /etc/ssh/sshd_config
```
```diff
- # Port 22
+ Port 12345
:wq
```
```bash
sudo semanage port -a -t ssh_port_t -p tcp 12345
sudo firewall-cmd --add-port=12345/tcp --permanent
sudo firewall-cmd --remove-service=ssh --permanent
```
- Install Caddy
<!-- TODO -->
