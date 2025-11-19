# Grafana Install (Bare-Metal)

Official Link: [Install Grafana on Debian or Ubuntu | Grafana documentation](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/)

Complete the following steps to install Grafana from the APT repository:

1. Install the prerequisite packages:
    
    ```bash
    sudo apt-get install -y apt-transport-https software-properties-common wget
    ```
    
2. Import the GPG key:
    
    ```bash
    sudo mkdir -p /etc/apt/keyrings/
    wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
    ```
    
3. To add a repository for stable releases, run the following command:
    
    ```bash
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    ```
    
4. To add a repository for beta releases, run the following command:
    
    ```bash
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    ```
    
5. Run the following command to update the list of available packages:
    
    ```bash
    # Updates the list of available packages
    sudo apt-get update
    ```
    
6. To install Grafana OSS, run the following command:
    
    ```bash
    # Installs the latest OSS release:
    sudo apt-get install grafana
    ```
    
7. To install Grafana Enterprise, run the following command:
    
    ```bash
    # Installs the latest Enterprise release:
    sudo apt-get install grafana-enterprise
    ```

# **Start the Grafana server**

## **Linux**

The following subsections describe three methods of starting and restarting the Grafana server: with systemd, initd, or by directly running the binary. You should follow only one set of instructions, depending on how your machine is configured.

### **Start the Grafana server with systemd**

Complete the following steps to start the Grafana server using systemd and verify that it is running.

1. To start the service, run the following commands:Bash
    
    ```jsx
    sudo systemctl daemon-reload
    sudo systemctl start grafana-server
    ```
    
2. To verify that the service is running, run the following command:Bash
    
    ```jsx
    sudo systemctl status grafana-server
    ```
    

### **Configure the Grafana server to start at boot using systemd**

To configure the Grafana server to start at boot, run the following command:

```jsx
sudo systemctl enable grafana-server.service
```
