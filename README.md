# Integrating Blumira's Cloud Based SIEM with a pfSense Firewall

## Objective

Welcome to my Blumira Integration Project! In this endeavor, I set up Blumira's dynamic blocklisting capabilities using pfBlockerNG on a pfSense firewall to enhance our network security. This project also involved deploying a Blumira sensor and honeypot to monitor and analyze potential threats effectively. It's an excellent hands-on experience for anyone looking to delve into network security and threat detection, especially those at the beginning of their IT careers.

### Skills Learned

Through this project, I gained valuable skills, including:

- **Deployment of Blumira Sensor and Honeypot**: Successfully set up a Blumira sensor on an Ubuntu server and configured it to function as a honeypot for enhanced security monitoring.
- **Integration with pfSense and pfBlockerNG**: Implemented dynamic blocklists in pfSense using pfBlockerNG, effectively filtering harmful traffic.
- **Monitoring Network Traffic**: Analyzed extensive network logs to understand traffic patterns and identify potential security incidents.
- **Vulnerability Generation and Testing**: Created intentional vulnerabilities to test the effectiveness of the security measures implemented.

### Tools Used

Here’s a quick overview of the tools and technologies employed in this project:

- **Blumira**: Cloud-based SIEM solution for monitoring and incident response.
- **pfSense**: Open-source firewall/router software to manage network traffic.
- **pfBlockerNG**: pfSense package for creating IP and DNS blocklists.
- **Ubuntu Server**: Operating system used to host the Blumira sensor.
- **Docker**: Containerization platform used for deploying the Blumira sensor.

## Table of Contents

1. [Introduction](#introduction)
2. [Setting Up the Ubuntu Server](#setting-up-the-ubuntu-server)
3. [Building and Installing the Blumira Sensor](#building-and-installing-the-blumira-sensor)
4. [Deploying the Honeypot](#deploying-the-honeypot)
5. [Configuring pfBlockerNG](#configuring-pfblockerng)
6. [Dynamic Blocklists](#dynamic-blocklists)
7. [Monitoring Network Traffic](#monitoring-network-traffic)
8. [Alerts Generated](#alerts-generated)
9. [Conclusion](#conclusion)

## Introduction

In this guide, I will walk you through the steps I took to integrate Blumira with our existing pfSense firewall. This project aimed to enhance our network security posture by implementing dynamic blocklisting and deploying a honeypot for threat detection.

## Setting Up the Ubuntu Server

To get started, I set up an Ubuntu server that would host the Blumira sensor. Here’s how I did it:

1. **Begin Installation**:
   - Follow the on-screen prompts to install Ubuntu Server with default settings.

2. **Choose Type of Install**:
   - Select Ubuntu Server and proceed with the installation.

3. **Configure Network Connections**:
   - Configure a static IP address using CIDR format (e.g., 192.168.1.0/24) to ensure consistent connectivity.

4. **Storage Configuration**:
   - Edit the storage configuration to utilize the entire disk and set it to the maximum usable size.

5. **SSH Setup**:
   - Select the option to install the OpenSSH server, allowing for remote access.

6. **Complete Installation**:
   - After the installation is complete, reboot the server.

## Building and Installing the Blumira Sensor

Once the Ubuntu server was up and running, I proceeded to install the Blumira sensor:

1. **Allowlist URLs**:
   - Ensure that the firewall permits outbound traffic to the URLs provided in the Blumira documentation.

2. **Log Into Ubuntu**:
   - Use an SSH client (e.g., PuTTY) to connect to the Ubuntu server with the credentials created during installation.

3. **Update System**:
   - Run the command: 
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

4. **Configure NTP Servers**:
   - Set NTP servers for time synchronization:
     ```bash
     NTP="0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org"
     sudo sed -ie "s/#*NTP=.*/NTP=$NTP/" /etc/systemd/timesyncd.conf
     sudo systemctl restart systemd-timesyncd
     ```

5. **Build and Install Sensor**:
   - In the Blumira dashboard, navigate to Settings > Sensors, and create a new sensor.
   - Copy the installation script generated for the Linux terminal and run it in the SSH session.
   - After installation, ensure that the Docker container is running, and reboot the sensor.

## Deploying the Honeypot

With the Blumira sensor in place, I configured it as a honeypot:

1. **Enable Honeypot Capability**:
   - Go to your sensor on the dashboard and enable the honeypot module

2. **Testing the Honeypot**:
   - In the browser, access the honeypot via the sensor's IP address and the designated port (e.g., `http://192.168.1.82:8080`).
   - Attempt to log in using invalid credentials, triggering an authentication failure alert.

## Configuring pfBlockerNG

Next, I configured pfBlockerNG on the pfSense firewall to implement dynamic blocklists:

1. **Access pfSense Web Interface**:
   - Open a browser and navigate to the pfSense interface.

2. **Install pfBlockerNG**:
   - Go to System > Package Manager > Available Packages, search for pfBlockerNG, and install it.

3. **Configure IP Blocklists**:
   - In pfBlockerNG, navigate to the IPv4 tab and add a new IP blocklist.
     - Description: Name the blocklist (e.g., Blumira_IP_Blocklist).
     - List URL: Enter the Blumira IP blocklist URL.
     - Set the update frequency and save the configuration.

4. **Configure DNSBL and URL Blocklists**:
   - Follow the same steps to configure DNS and URL blocklists using the URLs from Blumira.

## Dynamic Blocklists

After one month of using this SIEM, here are the dynamic blocklists generated:

- **IPv4 Blocklist**: [View IPv4 Blocklist](https://github.com/Jacob-Brown-950/pfSense-plus-SIEM/blob/main/IPv4%20Blocklist.txt)
- **Domain Blocklist**: [View Domain Blocklist](https://github.com/Jacob-Brown-950/pfSense-plus-SIEM/blob/main/DomainBlocklist.txt)
- **URL Blocklist**: [View URL Blocklist](https://github.com/Jacob-Brown-950/pfSense-plus-SIEM/blob/main/URL%20Blocklist.txt)

## Monitoring Network Traffic

After deploying the sensor and configuring the blocklists, I began monitoring network traffic:

1. **Traffic Analysis**:
   - I observed significant traffic from our public-facing Unifi networks, with the firewall forwarding around 500 million logs this month.

## Alerts Generated

To thoroughly test the security measures in place, I generated various alerts intentionally. The following activities triggered alerts in the system:

1. **Registry SAM Dumps**:
   - I executed SAM registry dumps on Windows devices to extract password hashes. This activity was detected by the SIEM, generating alerts indicating possible malicious attempts to access sensitive information.

2. **Nmap and Advanced IP Scanner Scans**:
   - Conducted port scans using Nmap and Advanced IP Scanner on the network. These scans revealed open ports and active services, which were logged and flagged as potential reconnaissance activities.

3. **Brute Force Attempts on Active Directory (AD) Account**:
   - I performed brute force attempts against a specific AD account, attempting to guess the password. The SIEM alerted me to the multiple failed login attempts, signaling possible credential stuffing or unauthorized access attempts.

4. **Storage of Fake Passwords**:
   - I created a text file containing fake passwords and accessed it from various machines on the network. This action raised alarms due to unusual behavior, as the file's access patterns did not align with normal user activities.

5. **Granting Unusual Permissions in Microsoft 365 Tenant**:
   - I altered permissions for another user in our Microsoft 365 tenant, granting them access that deviated from standard operational roles. This change was detected and flagged by the SIEM, indicating potential insider threats or misconfigurations.

## Conclusion

This project significantly enhanced our organization's security posture by integrating Blumira with our existing pfSense setup. Through deploying a sensor and honeypot, configuring dynamic blocklists, and actively monitoring network traffic, I gained valuable insights into threat detection and incident response.
