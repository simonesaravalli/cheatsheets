# Management of WSL2 instances

## Backup WSL2 instance

Check this guide: https://www.virtualizationhowto.com/2021/01/wsl2-backup-and-restore-images-using-import-and-export/

Basically, open a Powershell cli and run the following commands:

* List your WSL2 instances

```
wsl --list
```

* Stop the instance you need to backup

```
wsl --shutdown <instance name>
```

* Backup the WSL2 instance

```
wsl --export <Image Name> <Export location file name.tar>

ex

wsl --export ubuntu-2204 C:\Users\simonesaravalli\Desktop\ubuntu-2204.tar
```

## Restore WSL2 instance

To restore a WSL2 instance from a backup, run this command in Powershell:

```
wsl --import <Image Name> <Directory where you want to store the imported image> <Directory where the exported .tar file exists>
```

## Set memory limit of WSL2 instance

Check this guide: https://www.aleksandrhovhannisyan.com/blog/limiting-memory-usage-in-wsl-2/

* Create .wslconfig

Even directly within WSL instance, create or edit .wslconfig file:

```
editor "$(wslpath "C:\Users\YourUsername\.wslconfig")"
```

* Add the following configuration to set, for example, max 3 GB of RAM per WSL2 instance:

```
[wsl2]
memory=3GB
```

* Restart WSL instance to let memory change take effect

## Clone WSL2 instance

Check this guide: https://sungkim11.medium.com/why-you-should-use-multiple-instances-of-same-linux-distro-on-wsl-windows-10-f6f140f8ed88

Archived here: https://archive.ph/6SZDR