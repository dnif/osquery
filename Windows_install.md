### Installing osquery on Windows server

Osquey recomends to install and deploy Windows support using [`chocolatey`](https://chocolatey.org/).  
<b>Install command:</b>  
~~~~
C:\> choco.exe install osquery
~~~~  
For more details on chocolatey project, please visit [Chocolatey Project Page](https://chocolatey.org/packages/osquery).

##### How to install Chocolatey
* Requirments:  
  * Administrator Access
  * Windows 7+ / Windows Server 2003+
  * PowerShell v2+
  * .NET Framework 4+  
  [Note: the default installation attempts to install .NET 4.0 if you don't have it previously installed]

>**Note:** You need to run below mentioned commands as an **'Administrator'**.

<b>Installing Chocolatey Refference Document:</b>  
[https://chocolatey.org/install](https://chocolatey.org/install)

Using **'cmd.exe'** console:
```
C:> @powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

Output will look something like:
![Install_Choco][Pic_1]

Upgrade Chocolatey to latest version:
~~~~
C:> choco upgrade chocolatey
~~~~

Output:
![Upgrade_Choco][Pic_2]

<b>Installing osquery using chocolatey:</b>  
[https://chocolatey.org/packages/osquery](https://chocolatey.org/packages/osquery)

Installing `osquery` using **'choco.exe'** :
~~~~
C:> choco install osquery
~~~~

Output will look something like:
![Install_Osquery][Pic_3]

[Pic_1]: https://github.com/gauravtango/osquery/blob/master/Images/Win10_Install_Chocolaty.png
[Pic_2]: https://github.com/gauravtango/osquery/blob/master/Images/Win10_Chocolatey-Upgrade.png
[Pic_3]: https://github.com/gauravtango/osquery/blob/master/Images/win10_install_osquery.png

By default `choco.exe` chocolatey will install the binaries, example packs and configurations bundle to ``c:\ProgramData\osquery``. Yet at this point, `osqueryd` has been not installed as service.  
To install this, you can either pass Chocolatey the ``--params='/InstallService'`` flag during install command i.e.:
~~~~
C:\> choco.exe install osquery --params='/InstallService'
~~~~
<b>OR,</b>  
you can make use of osquery's `--install` flag with below given command to install `osqueryd` as a Windows system service.
~~~~
C:\ProgramData\osquery\osqueryd\osqueryd.exe --install
~~~~

![OSQry_as_Service_cmd][Pic_4]

You can also create `osquery-service-daemon` using Powershell `manage-osqueryd.ps1` script in `C:\ProgramData\osquery` folder.
This same script can also be used to <b>start</b> and <b>stop</b> the `osqueryd` service.  

![PS_manage_service_cmd][Pic_5]

[Pic_4]: https://github.com/gauravtango/osquery/blob/master/Images/Win10-install-osqueryd-as-service.png  
[Pic_5]: https://github.com/gauravtango/osquery/blob/master/Images/Win10_install_daemon_manageService_PS.png

---
### Creating osquery configuration file
Creating configuration file makes it easier to run <code>osqueryi</code> and <code>osqueryd</code>. Insted of passing lots of command line options, both <code>osqueryi</code> and <code>osqueryd</code> can read those options from a configuration file from file-path <code>C:\ProgramData\osquery\osquery.conf</code>.

<b>There are three sections to the configuration file:</b>  
* A list of daemon service options
* A list of scheduled queries to run and when they should run.
* A list of packs to include more specific scheduled queries.

Create and open the configuration file at:
<code>C:\ProgramData\osquery\osquery.conf</code>
>Note: make sure that created file had proper extention as `osquery.conf` and not `osquery.conf.txt`.

The configuration file uses the JSON format. Copy the following content into the file:  
```css
{
  "options": {
    "config_plugin": "filesystem",
    "logger_plugin": "filesystem",
    "disable_logging": "false",
    "log_result_events": "true",
    "schedule_splay_percent": "10",
    "events_expiry": "3600",
    "verbose": "false",
    "worker_threads": "2",
    "enable_monitor": "true",
    "disable_events": "false",
    "disable_audit": "false",
    "audit_allow_config": "true",
    "host_identifier": "hostname",
    "enable_syslog": "false",
    "audit_allow_sockets": "true",
    "schedule_default_interval": "3600"
  },
  "OSVer": {
    "crontab": {
      "query": "SELECT * FROM os_version;",
      "interval": 3600
    },
    "OSQSysProfile": {
      "query": "SELECT * FROM osquery_schedule;",
      "interval": 3600
    },
    "SysInfo": {
      "query": "SELECT * FROM system_info;",
      "interval": 3600
    }
  },
  "decorators": {
    "load": [
      "SELECT uuid AS host_uuid FROM system_info;",
      "SELECT user AS username FROM logged_in_users ORDER BY time DESC LIMIT 1;"
    ]
  },
  "packs": {
     "osquery-monitoring": "/usr/share/osquery/packs/osquery-dnif-linux.conf"
  }
}
```
Save and close the file, then validate it using the following <b>PowerShell</b> command:  
`` PS C:\ProgramData\osquery\osqueryd> .\osqueryd.exe -config_check ``
If there's an error, the output will indicate the location of the error so you can fix it.

---  
### Running osqueryd

Once the configuration file is in place, we can start osqueryd using any of below given methods:  

1. Using **cmd.exe**  
``Start-Service osqueryd``

2. Using **PowerShell**  
``sc.exe start osqueryd``
**OR**
``PS C:\ProgramData\osquery> .\manage-osqueryd.ps1 -start ``

3. Using **Run-->service.msc**  
Locate service <b>'osquery service daemon'</b> and start it.

![service_msc_SShot][Pic_6]

[Pic_6]: https://github.com/gauravtango/osquery/blob/master/Images/Win10_Osquery_daemon_start.png
