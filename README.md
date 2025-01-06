# Auto-back-up
import os
import paramiko
from scp import SCPClient
from datetime import datetime

def create_scp_client(hostname, port, username, password):
    """Create an SCP client."""
    try:
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(hostname, port=port, username=username, password=password)
        return SCPClient(ssh.get_transport())
    except Exception as e:
        print(f"Failed to connect to the server: {e}")
        return None

def backup_directory(source_directory, remote_directory, hostname, port, username, password):
    """Backup the specified directory to a remote server."""
    success = False
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    report = f"Backup Report for {source_directory}:\n"

    if not os.path.exists(source_directory):
        report += f"{timestamp} - Source directory does not exist: {source_directory}\n"
        return report

    try:
        scp = create_scp_client(hostname, port, username, password)
        if not scp:
            report += f"{timestamp} - Failed to create SCP client.\n"
            return report

        scp.put(source_directory, recursive=True, remote_path=remote_directory)
        success = True
        report += f"{timestamp} - Backup of {source_directory} to {remote_directory} succeeded.\n"
    except Exception as e:
        report += f"{timestamp} - Backup failed: {e}\n"
    finally:
        if scp:
            scp.close()

    return report

if __name__ == "__main__":
    # Configuration
    source_directory = "/path/to/source/directory"
    remote_directory = "/path/to/remote/backup/directory"
    hostname = "remote.server.com"
    port = 22
    username = "your_username"
    password = "your_password"  # Use key-based authentication for better security

    # Perform the backup
    report = backup_directory(source_directory, remote_directory, hostname, port, username, password)

    # Print the report
    print(report)
