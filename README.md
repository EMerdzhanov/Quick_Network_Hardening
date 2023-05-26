#!/bin/bash

# Enable firewall
echo "Enabling firewall..."
ufw enable

# Configure firewall rules
echo "Configuring firewall rules..."
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh/tcp
ufw allow http/tcp
ufw allow https/tcp
ufw allow ftp/tcp

# Harden SSH configuration
echo "Hardening SSH configuration..."
sed -i 's/#Port 22/Port 2222/g' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
service ssh restart

# Configure user and group permissions
echo "Configuring user and group permissions..."
chmod 750 /etc/shadow
chmod 750 /etc/passwd
chmod 750 /etc/group

# Disable root login
echo "Disabling root login..."
usermod -p '!' root

# Harden network configuration
echo "Hardening network configuration..."
sed -i 's/#net.ipv4.tcp_syncookies=1/net.ipv4.tcp_syncookies=1/g' /etc/sysctl.conf
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=0/g' /etc/sysctl.conf
sysctl -p

# Install and configure fail2ban
echo "Installing and configuring fail2ban..."
apt-get install fail2ban
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sed -i 's/bantime  = 600/bantime  = 3600/g' /etc/fail2ban/jail.local
sed -i 's/findtime  = 600/findtime  = 300/g' /etc/fail2ban/jail.local
sed -i 's/maxretry = 3/maxretry = 5/g' /etc/fail2ban/jail.local
service fail2ban restart

# Harden Apache web server
echo "Hardening Apache web server..."
cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf.bak
sed -i 's/ServerTokens OS/ServerTokens Prod/g' /etc/apache2/apache2.conf
sed -i 's/ServerSignature On/ServerSignature Off/g' /etc/apache2/apache2.conf
service apache2 restart

echo "System hardening complete."

#This script performs a variety of hardening tasks,
#including enabling and configuring the firewall,
#hardening SSH configuration, configuring user and group permissions,
#disabling root login, hardening network configuration,
#installing and configuring fail2ban,
#and hardening the Apache web server configuration.

