# web-dev-on-cloud-script
Quick script to start doing web dev on cloud servers
```python3
import subprocess
import os

subprocess.run(["curl", "-fsSL", "https://deb.nodesource.com/setup_18.x", "|", "sudo", "-E", "bash", "-"])
subprocess.run(["sudo", "apt-get", "install", "-y", "nodejs"])

subprocess.run(["sudo", "apt-get", "install", "-y", "build-essential", "tor"])

subprocess.run(["curl", "-fsSL", "https://code-server.dev/install.sh", "|", "sh"])
subprocess.run(["sudo", "systemctl", "stop", "code-server"])
subprocess.run(["sudo", "systemctl", "enable", "--now", "code-server"])

subprocess.run(["sed", "-i", "s/8080/8220/g", "/etc/code-server/config.yaml"])
password = subprocess.run(["sh", "-c", "cat /home/$USER/.config/code-server/config.yaml | grep 'password' | awk '{print $2}'"], capture_output=True, text=True).stdout.strip()
print(password)

subprocess.run(["sudo", "apt-get", "install", "-y", "tor"])
os.makedirs("/var/lib/tor/hidden_service_8220", exist_ok=True)
os.makedirs("/var/lib/tor/hidden_service_5173", exist_ok=True)
with open("/etc/tor/torrc", "a") as f:
    f.write("\nHiddenServiceDir /var/lib/tor/hidden_service_8220/\nHiddenServicePort 80 127.0.0.1:8220\n")
    f.write("HiddenServiceDir /var/lib/tor/hidden_service_5173/\nHiddenServicePort 80 127.0.0.1:5173\n")

subprocess.run(["sudo", "systemctl", "restart", "tor"])

# Wait for files to exist before printing hostname
while not os.path.exists("/var/lib/tor/hidden_service_8220/hostname"):
    pass
print("Onion service for code-server: ", subprocess.run(["sudo", "cat", "/var/lib/tor/hidden_service_8220/hostname"], capture_output=True, text=True).stdout.strip())

while not os.path.exists("/var/lib/tor/hidden_service_5173/hostname"):
    pass
print("Onion service for port 5173: ", subprocess.run(["sudo", "cat", "/var/lib/tor/hidden_service_5173/hostname"], capture_output=True, text=True).stdout.strip())

# Restart code-server
subprocess.run(["sudo", "systemctl", "restart", "code-server"])
```
