# SatisfactoryServerInstall

## Basics

```
passwd
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
```

## Powertop
```
cat << EOF | sudo tee /etc/systemd/system/powertop.service
[Unit]
Description=Powertop tunings

[Service]
Type=oneshot
Environment="TERM=dumb"
RemainAfterExit=yes
ExecStart=/usr/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
EOF
```

## Screensaving
```
cat << EOF | sudo tee /etc/systemd/system/screenoff.service
[Unit]
Description=Screen off

[Service]
Type=oneshot
Environment="TERM=linux"
RemainAfterExit=yes
ExecStart=/usr/bin/setterm -blank 1
StandardOutput=tty
TTYPath=/dev/console

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable powertop.service
sudo systemctl enable screenoff.service
sudo reboot
```

## SSH keys
```
mkdir ~/.ssh
echo "XXXX PUBLIC KEY" > ~/.ssh/authorized_keys
sed -i 's/^#ChallengeResponseAuthentication.*/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^UsePAM.*/UsePAM no/' /etc/ssh/sshd_config
sudo reboot
```

## Steam install
```
sudo useradd -m steam
sudo passwd steam
```

Enter password

```
sudo usermod -a -G sudo steam
sudo -u steam -s
cd /home/steam

sudo add-apt-repository multiverse; sudo dpkg --add-architecture i386; sudo apt update
sudo apt install steamcmd

sudo ln -s /usr/games/steamcmd /home/steam/steamcmd
export PATH=$PATH:/usr/games
source $HOME/.profile

steamcmd +force_install_dir ~/SatisfactoryDedicatedServer +login anonymous +app_update 1690800 -beta public validate +quit
```

## Simple start
```
cd ~/SatisfactoryDedicatedServer/
./FactoryServer.sh
```

## Service
```
cat << EOF | sudo tee /etc/systemd/system/satisfactory.service
[Unit]
Description=Satisfactory dedicated server
Wants=network-online.target
After=syslog.target network.target nss-lookup.target network-online.target

[Service]
Environment="LD_LIBRARY_PATH=./linux64"
ExecStartPre=/usr/games/steamcmd +force_install_dir "/home/steam/SatisfactoryDedicatedServer" +login anonymous +app_update 1690800 validate +quit
ExecStart=/home/steam/SatisfactoryDedicatedServer/FactoryServer.sh
User=steam
Group=steam
StandardOutput=journal
Restart=always
WorkingDirectory=/home/steam

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable satisfactory
sudo systemctl start satisfactory
```
