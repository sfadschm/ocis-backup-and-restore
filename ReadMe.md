# oCIS Backup and Restore

## What?

This gist intends to provide a starting point for all those who (like me) want to use ownCloud Infinite Scale (oCIS) for a simple private cloud deployment and are puzzled by how to setup automatic backups of your data that can be restored later.

Unfortunately, the oCIS documentation (unlike ownCloud 10) is pretty sparse with useful information.

The main issue when backing up an oCIS deployment and especially when trying to restore individual files is it's underlying decomposed file system.
At the current stage of oCIS development, this means that only opaque backups are possible and from discussions in the official ownCloud channels it seems questionable whether this will change in the future.

The scripts included in this gist can handle the following scenario:

- oCIS runs as a docker deployment on a Linux / Ubuntu server
- a Synology DiskStation (DS) is used as a backup target
- the backup is to be restored on a local machine (e.g., in a Windows Subsystem for Linux - WSL)

While the scripts have not been tested in other deployments, adaptions (e.g., leaving out the intermediate backup location) should be straightforward.

## How?

This gist contains two *bash* scripts, namely `backup` and `restore`, which carry the following tasks:

- `backup` (this can, e.g. be used for automatic backups using a task scheduler)
    - stop the oCIS live container
    - pull all crucial oCIS directories (i.e., the *container*, *ocis-config* and *ocis-data*) via `rsync` (incremental backup)
    - restart the live container
- `restore`
    - pull the latest full backup from the backup source to the local machine
    - extract the oCIS version from the container and pull the binary executable
    - start up the oCIS binary server

If anything goes well, you will see a browser pop up with your `localhost` open which prompts you with the oCIS login form.
From there on, you can login and play around with your files just as in the live deployment.

## Prerequisites

To replicate the original use case you need:

- a remote server with
    - Ubuntu or any other Linux-based OS
    - an [oCIS docker deployment](https://owncloud.dev/ocis/deployment/ocis_hello/)
    - `rsync` installed
    - `ssh` access (preferably via RSA key pair for automation)
- a remote or local Synology DiskStation as backup target with
    - [`ssh` enabled](https://kb.synology.com/de-de/DSM/tutorial/How_to_login_to_DSM_with_root_permission_via_SSH_Telnet)
    - [`rsync` enabled](https://kb.synology.com/de-de/DSM/help/DSM/AdminCenter/file_rsync?version=7)
    - a dedicated shared folder to store the backup in (e.g., ownCloud)
- a local computer (or server) for restoring the data with
    - a Linux-based OS or a WSL

## Usage

You must fill in the `XXX` voids in the parameters of each script. Depending on you setup, our will also need to adjust the `rsync` access port and user in the `restore` script.

You can either run each script manually via the corresponding terminal on the respective machine or automate the `backup` command, e.g. by using the [DS Task Scheduler](https://kb.synology.com/en-nz/DSM/help/DSM/AdminCenter/system_taskscheduler?version=7).

The `restore` script comes with two command line flags so you do not need to pull all data again only because the server crashed:
- '-b' will pull the oCIS binary (e.g., if you changed the version string in-between restores)
- '-p' will pull the backup data
- if no flag is set, this will only try to start the server (which will fail, if you did not at least once call the script with each of the above flags set)

## Disclaimer

Use at your own risk.
Suggestions and improvements welcome at any time.
