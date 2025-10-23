# cloudwatch-agent-setup-in-instance
# Amazon CloudWatch Agent Installation & Configuration
## Complete Command Documentation

---

## Command 1: System Package Update

### Command
```bash
sudo apt update -y
```

### Purpose
Updates the local package index with the latest changes made in repositories. This ensures you can install the most recent versions of packages.

### Use Case
- Before installing any new software on Ubuntu/Debian systems
- Regular system maintenance
- Ensuring security patches are available

### Command Breakdown
- **`sudo`** - Execute command with superuser (root) privileges
- **`apt`** - Advanced Package Tool, package management system for Debian/Ubuntu
- **`update`** - Synchronizes package index files from their sources
- **`-y`** - Automatically answer "yes" to all prompts

### Expected Output
```
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
Reading package lists... Done
```

---

## Command 2: Install Required Dependencies

### Command
```bash
sudo apt install -y curl unzip
```

### Purpose
Installs curl (command-line tool for transferring data) and unzip (archive extraction utility) which are required for downloading and extracting the CloudWatch agent.

### Use Case
- Installing tools needed to download files from the internet
- Preparing system for software installation from remote sources
- Essential utilities for automation scripts

### Command Breakdown
- **`sudo`** - Run with root privileges
- **`apt install`** - Install packages
- **`-y`** - Auto-confirm installation without prompting
- **`curl`** - Tool to transfer data from/to a server using various protocols
- **`unzip`** - Utility to extract compressed .zip archives

### Expected Output
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  curl unzip
0 upgraded, 2 newly installed, 0 to remove
```

---

## Command 3: Change to Temporary Directory

### Command
```bash
cd /tmp
```

### Purpose
Changes current working directory to /tmp, a standard location for temporary files that are typically cleared on system reboot.

### Use Case
- Storing temporary download files
- Preventing clutter in home or system directories
- Working with files that don't need to persist across reboots

### Command Breakdown
- **`cd`** - Change Directory command
- **`/tmp`** - Temporary directory path (absolute path from root)

### Expected Output
No output (silent success). Use `pwd` to verify current directory.

---

## Command 4: Download CloudWatch Agent

### Command
```bash
curl -O https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
```

### Purpose
Downloads the latest Amazon CloudWatch agent installation package (.deb file) from AWS S3 bucket to the current directory.

### Use Case
- Obtaining official AWS CloudWatch agent software
- Downloading large files from web sources
- Automated software deployment scripts

### Command Breakdown
- **`curl`** - Command-line data transfer tool
- **`-O`** - Save file with remote filename (capital O)
- **URL** - Direct link to CloudWatch agent package for Ubuntu 64-bit systems

### Alternative Options
- `-o filename` - Save with custom filename
- `-L` - Follow redirects if URL has moved
- `--progress-bar` - Show download progress

### Expected Output
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 45.2M  100 45.2M    0     0  12.3M      0  0:00:03  0:00:03 --:--:-- 12.3M
```

---

## Command 5: Install CloudWatch Agent Package

### Command
```bash
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

### Purpose
Installs the CloudWatch agent .deb package on the system using Debian package manager.

### Use Case
- Installing software from .deb packages
- System-level software installation on Debian/Ubuntu
- Deploying AWS monitoring agents on EC2 instances

### Command Breakdown
- **`sudo`** - Execute with administrative privileges
- **`dpkg`** - Debian Package manager
- **`-i`** - Install package flag
- **`-E`** - Don't overwrite existing configuration files
- **`./amazon-cloudwatch-agent.deb`** - Path to the package file

### Expected Output
```
Selecting previously unselected package amazon-cloudwatch-agent.
(Reading database ... 123456 files and directories currently installed.)
Preparing to unpack amazon-cloudwatch-agent.deb ...
Unpacking amazon-cloudwatch-agent ...
Setting up amazon-cloudwatch-agent ...
```

### Installation Location
- Binary: `/opt/aws/amazon-cloudwatch-agent/bin/`
- Config: `/opt/aws/amazon-cloudwatch-agent/etc/`
- Logs: `/opt/aws/amazon-cloudwatch-agent/logs/`

---

## Command 6: Check Agent Status

### Command
```bash
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
```

### Purpose
Verifies the installation and checks the current operational status of the CloudWatch agent.

### Use Case
- Confirming successful installation
- Troubleshooting agent issues
- Monitoring agent health in automation scripts

### Command Breakdown
- **Full path** - Direct execution of agent control script
- **`amazon-cloudwatch-agent-ctl`** - Agent control utility
- **`-a status`** - Action parameter set to check status

### Expected Output (Not Running)
```
{
  "status": "stopped",
  "starttime": "",
  "version": "1.300025.0"
}
```

### Expected Output (Running)
```
{
  "status": "running",
  "starttime": "2025-10-23T10:30:45+0000",
  "version": "1.300025.0"
}
```

---

## Command 7: Create Configuration File

### Command
```bash
sudo tee /opt/aws/amazon-cloudwatch-agent/bin/config.json > /dev/null <<'EOF'
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          "used_percent"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "/"
        ]
      }
    }
  }
}
EOF
```

### Purpose
Creates a JSON configuration file that defines what metrics the CloudWatch agent should collect and how frequently.

### Use Case
- Configuring custom metric collection
- Defining monitoring parameters for EC2 instances
- Standardizing monitoring across multiple servers

### Command Breakdown
- **`sudo tee`** - Write stdin to file with elevated privileges
- **File path** - `/opt/aws/amazon-cloudwatch-agent/bin/config.json`
- **`> /dev/null`** - Discard stdout (suppress terminal output)
- **`<<'EOF'`** - Here-document delimiter (single quotes prevent variable expansion)

### Configuration Explanation

#### Agent Section
```json
"agent": {
  "metrics_collection_interval": 60,  // Collect every 60 seconds
  "run_as_user": "root"                // Run agent as root user
}
```

#### Metrics Section
```json
"append_dimensions": {
  "InstanceId": "${aws:InstanceId}"    // Auto-tag with EC2 Instance ID
}
```

#### Memory Metrics
```json
"mem": {
  "measurement": ["mem_used_percent"], // Track memory usage %
  "metrics_collection_interval": 60    // Check every 60 seconds
}
```

#### Disk Metrics
```json
"disk": {
  "measurement": ["used_percent"],     // Track disk usage %
  "metrics_collection_interval": 60,
  "resources": ["/"]                   // Monitor root partition
}
```

### Available Metrics
- **Memory**: `mem_used_percent`, `mem_available`, `mem_total`
- **Disk**: `used_percent`, `free`, `total`, `used`
- **CPU**: `cpu_usage_idle`, `cpu_usage_active`
- **Network**: `bytes_sent`, `bytes_recv`, `packets_sent`

---

## Command 8: Load Configuration and Start Agent

### Command
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

### Purpose
Applies the configuration file to the CloudWatch agent and starts the monitoring service.

### Use Case
- Activating CloudWatch agent with custom configuration
- Deploying monitoring to EC2 instances
- Reloading configuration after changes

### Command Breakdown
- **`-a fetch-config`** - Action: fetch and apply configuration
- **`-m ec2`** - Mode: EC2 environment (vs on-premises)
- **`-c file:PATH`** - Configuration source with file path
- **`-s`** - Start the agent after loading configuration

### Alternative Modes
- **`-m ec2`** - For EC2 instances
- **`-m onPremise`** - For on-premises servers
- **`-m auto`** - Auto-detect environment

### Expected Output
```
****** processing amazon-cloudwatch-agent ******
Successfully fetched the config and saved in /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/file_config.json
Start configuration validation...
Configuration validation succeeded
amazon-cloudwatch-agent has been successfully started
```

---

## Command 9: Verify Service Status

### Command
```bash
sudo systemctl status amazon-cloudwatch-agent
```

### Purpose
Checks the systemd service status of CloudWatch agent, showing whether it's active, recent logs, and process information.

### Use Case
- Verifying agent is running properly
- Viewing recent error messages
- Troubleshooting service issues
- Confirming auto-start configuration

### Command Breakdown
- **`sudo`** - Run with elevated privileges
- **`systemctl`** - systemd system and service manager
- **`status`** - Show service status information
- **`amazon-cloudwatch-agent`** - Service name

### Expected Output (Running)
```
● amazon-cloudwatch-agent.service - Amazon CloudWatch Agent
     Loaded: loaded (/etc/systemd/system/amazon-cloudwatch-agent.service; enabled)
     Active: active (running) since Thu 2025-10-23 10:30:45 UTC; 2h ago
   Main PID: 12345 (amazon-cloudwat)
      Tasks: 8 (limit: 4915)
     Memory: 45.2M
     CGroup: /system.slice/amazon-cloudwatch-agent.service
             └─12345 /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent

Oct 23 10:30:45 ip-172-31-1-1 systemd[1]: Started Amazon CloudWatch Agent.
```

### Related systemctl Commands
- **`systemctl start`** - Start the service
- **`systemctl stop`** - Stop the service
- **`systemctl restart`** - Restart the service
- **`systemctl enable`** - Enable auto-start on boot
- **`systemctl disable`** - Disable auto-start

---

## Additional Information

### Prerequisites
1. **IAM Role**: EC2 instance must have IAM role with `CloudWatchAgentServerPolicy`
2. **Network**: Outbound HTTPS (443) access to CloudWatch endpoints
3. **OS**: Ubuntu/Debian-based Linux distribution

### Important File Locations
- **Agent Binary**: `/opt/aws/amazon-cloudwatch-agent/bin/`
- **Configuration**: `/opt/aws/amazon-cloudwatch-agent/etc/`
- **Logs**: `/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log`
- **systemd Service**: `/etc/systemd/system/amazon-cloudwatch-agent.service`

### Troubleshooting Commands
```bash
# View real-time logs
sudo tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log

# Check configuration validation
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a query-config \
  -m ec2 \
  -c default

# Restart agent
sudo systemctl restart amazon-cloudwatch-agent

# Check if metrics are being sent
aws cloudwatch list-metrics --namespace CWAgent
```

### Security Best Practices
1. Use IAM roles instead of access keys
2. Apply principle of least privilege to IAM policies
3. Regularly update the CloudWatch agent
4. Monitor agent logs for errors
5. Use encrypted CloudWatch log groups

### Cost Considerations
- **Metrics**: Charged per custom metric
- **API Requests**: GetMetricStatistics calls incur costs
- **Log Storage**: CloudWatch Logs storage is charged per GB
- **Free Tier**: 10 custom metrics and 5GB log ingestion/month

---

## Quick Reference

| Command | Purpose | Output |
|---------|---------|--------|
| `apt update` | Update package lists | Package sync status |
| `apt install` | Install packages | Installation progress |
| `dpkg -i` | Install .deb package | Package setup status |
| `tee` | Write to file | File creation |
| `systemctl status` | Check service status | Service state & logs |

---

**Document Version**: 1.0  
**Last Updated**: October 23, 2025  
**AWS CloudWatch Agent Version**: Latest (1.300025.0+)  
**Supported OS**: Ubuntu 18.04, 20.04, 22.04, 24.04

---

## References
- AWS CloudWatch Agent Documentation: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html
- Ubuntu APT Guide: https://help.ubuntu.com/community/AptGet/Howto
- systemd Manual: https://www.freedesktop.org/software/systemd/man/systemctl.html
