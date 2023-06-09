# SatisfactoryServerInstall

## Basics

```
passwd
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
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

sudo add-apt-repository multiverse
sudo apt install software-properties-common
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install lib32gcc-s1 steamcmd

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

## Add swap space
```
to do
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
