# Upgrade Percona Backup for MongoDB

Similar to installing, the recommended and most convenient way to upgrade PBM is from the Percona repository.

You can upgrade Percona Backup for MongoDB either to the **latest version** or to a **specific version**. Since all packages of Percona Backup for MongoDB are stored in the same repository, the following steps apply to both upgrade scenarios:

1. Enable Percona repository.
2. Stop `pbm-agent`.
3. Install new version packages (the old ones are automatically removed).
4. Start `pbm-agent`.

## Important notes

1. Backward compatibility between data backup and restore is supported for upgrades within one major version only (for example, from 1.1.x to 1.2.y). When you upgrade Percona Backup for MongoDB over several major versions (for example, from 1.0.x to 1.2.y), we recommend to make a backup right after the upgrade.

2. Percona Backup for MongoDB v1.5.0 and later is incompatible with Percona Backup for MongoDB v1.4.1 and earlier due to different processing of system collections `Users` and `Roles` during backup / restore operations. After the upgrade to Percona Backup for MongoDB v1.5.0 and later, make sure to make a fresh backup.

3. Starting from v1.7.0, the user running the `pbm-agent` process is changed from `pbm` to `mongod`. This is done for the following reasons:

    * To make physical backups and restores, the user running the `pbm-agent` process must have the read / write permissions to the MongoDB `dataDir`.

    * To use the filesystem-based backup storage, the user running the `pbm-agent` process must also have the read / write permissions to the backup directory.

4. Starting from version 1.3.0, Percona Backup for MongoDB packages are stored in the `pbm` repository and the `tools` repository for backward compatibility.

5. Upgrade Percona Backup for MongoDB on all nodes where it is installed.

!!! important 

    Run all commands as root or via `sudo`.

## Prerequisites

### 1. Install `percona-release`

[Install `percona-release` tool](https://www.percona.com/doc/percona-repo-config/installing.html). If you have installed it before, [update](https://www.percona.com/doc/percona-repo-config/updating.html) it to the latest version.

### 2. Enable the repository

Run the following command as root or via `sudo`

```sh
sudo percona-release enable pbm release
```

!!! note

    For `apt`-based systems, run `sudo apt update` to update the local cache.

## Upgrade to the latest version

=== "On Debian and Ubuntu Linux"

    ### 1. Stop `pbm-agent`

      ```sh
      sudo systemctl stop pbm-agent
      ```

    ### 2. Install new packages

      ```sh
      sudo apt install percona-backup-mongodb
      ```  

    ### 3. Reload the `systemd` process

    Starting from v1.7.0, reload the `systemd` process to update the unit file with the following command:

     ```sh
     sudo systemctl daemon-reload
     ```

    ### 4. Update permissions

    For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.


    ### 5. Start `pbm-agent`

      ```sh
      sudo systemctl start pbm-agent
      ```

=== "On Red Hat Enterprise Linux and derivatives"

    ### 1. Stop `pbm-agent`

      ```sh
      sudo systemctl stop pbm-agent
      ```

    ### 2. Install new packages

      ```sh
      sudo yum install percona-backup-mongodb
      ```

    ### 3. Reload the `systemd` process

    Starting from v1.7.0, reload the `systemd` process to update the unit file with the following command:

     ```sh
     sudo systemctl daemon-reload
     ```

    ### 4. Update permissions

    For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.


    ### 5. Start `pbm-agent`

      ```sh
      sudo systemctl start pbm-agent
      ``` 

## Upgrade to a specific version

=== "On Debian and Ubuntu Linux"

    ### 1. List available versions

     ```sh
     sudo apt-cache madison percona-backup-mongodb
     ```

    Output:

     ```text
     percona-backup-mongodb | 1.8.1-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
     percona-backup-mongodb | 1.8.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
     percona-backup-mongodb | 1.7.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
     percona-backup-mongodb | 1.6.1-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
     percona-backup-mongodb | 1.6.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
     percona-backup-mongodb | 1.5.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
     ```

    ### 2. Stop `pbm-agent`

       ```sh
       sudo systemctl stop pbm-agent
       ```

    ### 3. Install packages

    Install a specific version packages. For example, to upgrade to Percona Backup for MongoDB 1.7.0, run the following command:

     ```sh
     sudo apt install percona-backup-mongodb=1.7.0-1.stretch
     ```
 
    ### 4. Update permissions

    For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.


    ### 5. Start `pbm-agent`

      ```sh
      sudo systemctl start pbm-agent
      ``` 

=== "On Red Hat Enterprise Linux and derivatives"
  
    ### 1. List available versions

     ```sh
     sudo yum list percona-backup-mongodb --showduplicates
     ```

    Output:

     ```text
     Available Packages
     percona-backup-mongodb.x86_64    1.8-1.el7            pbm-release-x86_64
     percona-backup-mongodb.x86_64    1.8.0-1.el7          pbm-release-x86_64
     percona-backup-mongodb.x86_64    1.7.0-1.el7          pbm-release-x86_64
     percona-backup-mongodb.x86_64    1.6.1-1.el7          pbm-release-x86_64
     percona-backup-mongodb.x86_64    1.6.0-1.el7          pbm-release-x86_64
     percona-backup-mongodb.x86_64    1.5.0-1.el7          pbm-release-x86_64
     ```

    ### 2. Stop `pbm-agent`

       ```sh
       sudo systemctl stop pbm-agent
       ```

    ### 3. Install packages

    Install a specific version packages. For example, to upgrade to Percona Backup for MongoDB 1.7.1, run the following command:

     ```sh
     sudo yum install percona-backup-mongodb-1.7.1-1.el7
     ```
    
    ### 4. Update permissions

    For a *filesystem-based backup storage*, grant read / write permissions to the backup directory to the `mongod` user.


    ### 5. Start `pbm-agent`

      ```sh
      sudo systemctl start pbm-agent
      ``` 

!!! note

    If MongoDB runs under a *different user than `mongod`* (the default configuration for Percona Server for MongoDB), use the same user to run the `pbm-agent`. For filesystem-based storage, grant the read / write permissions to the backup directory for this user.
