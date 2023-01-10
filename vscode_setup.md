### Installation of VS Code and setting up the remote connections

* Install openssh: depending on your windows version you
  * can install this using the [install optional features](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse) feature of windows or,
  * have to install it directly from [github](https://github.com/PowerShell/Win32-OpenSSH/releases), see some [instructions here](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH).
* Install [VS Code](https://code.visualstudio.com/)
* Run VS Code, go to View > Extensions and install the Remote Development extension
* Get acquainted with the [remote-ssh extension](https://code.visualstudio.com/docs/remote/ssh)
  * [SSH host setup](https://code.visualstudio.com/docs/remote/ssh#_ssh-host-setup)
  * For Windows, follow the steps described [here](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
  * For PuTTY users: [simpler way available by converting your PuTTY SSH key into an OpenSSH key](https://www.thegeekdiary.com/how-to-convert-puttys-private-key-ppk-to-ssh-key/)
* To connect to a remote location press `F1` and run `Remote-SSH: Connect to Host`
* Specify the remote with `ssh user@hostname` for example `ssh user@aix.pik-potsdam.de` or `cluster.pik-potsdam.de` (to directly connect to the cluster you need to adjust your config file manually).
* Select as the operating system of the remote (for the cluster `Linux`)
* Enter your passphrase/password (depending on the target) to connect
* To connect to the cluster open your config.js by pressing F1 and calling `Remote-SSH: Open SSH Configuration File` and select the respective file. Add the following connection:
```
Host cluster.pik-potsdam.de
  HostName cluster.pik-potsdam.de
  User <username>
  ProxyJump aix.pik-potsdam.de
```
* **Note:** With the current version of VS Code you do not need the ProxyJump anymore.
* Connect to the cluster.

#### Troubleshooting

* If you get the following error `"Could not establish connection to cluster.pik-potsdam.de"`, try [generating a new public key with PuttyGen](https://code.visualstudio.com/docs/remote/troubleshooting#_reusing-a-key-generated-in-puttygen): e.g. `C:\Users\user\.ssh\id_dsa.pub`. Then, add the path to the config file.
```
Host cluster.pik-potsdam.de
  HostName cluster.pik-potsdam.de
  User <username>
  IdentityFile C:\Users\user\.ssh\id_dsa.pub
```
* I experienced issues with orphan processes rendering it impossible to run anything, this can, unfortunately, be caused by several things:
  * VS Code is running a lot of extensions on the remote: to prevent this, if you install new extensions always check where they are installed and run. On the left column, brows to the Extension icon and check if the extension is under LOCAL or SHH (cluster). Eventually, try to disable the extension you do not need at the moment.
  * The  typescript and javascript language features extension can also lead to the creation of a lot of processes see [here](https://github.com/microsoft/vscode-remote-release/issues/262). To clean up processes, try the following:
    * display current processes: `top -u <user>`
    * find `cpptools` processe IDs (PID): `pgrep cpptools`
    * kill processes `kill <PID>`
    * in alternative to kill all cpptools processes: `killall -u <user> cpptools`
    * If processes are zombie processes they are not necessarily killed with `killall`. If this is the case run `ps -A -ostat,ppid | grep -e '[zZ]'| awk '{ print $2 }' | uniq | xargs ps -p` to identify the parent processes of all zombies (be carefull this shows the zombies for all users). The first columns shows the parent process id (ppid). Execute `kill -9 <ppid>` for all processes you want to kill.
    * Another reason why processes don't get killed is because you are trying to kill them from another login node. In this case, switch between login nodes via `ssh login01` or `ssh login02` and run the kill commands as above.
* It can happen that an update of VS code starts automatically and crashes. The weird thing is that the program apparently uninstalls itself and simply disappears. I have no idea why this happened, but just downloading the latest version and reinstalling it solved the issue and importantly, all the settings I had defined before were recovered. So, don't panic and reinstall :wink:!
