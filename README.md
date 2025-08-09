# CloudFlare Dynamic VPS Updater
![DucksFeet](ducksfeet-left-100.png "DucksFeet Logo")
For persons with a dynamic IP address, usually as a residential broadband customer, this tool will allow 
for updating dynamic DNS on Cloudflare to assist with that. 

*Note: Residential Internet Service Providers may have terms of service that do not allow for running services over the IP address they give you.*

This tool is meant to be run systemd or cron(8). Included are two systemd files, a timer and a service unit. This allows for better logging and overall control.

Rather than be a quick, one host, one shot tool, this aims to offer great flexibility and control over 
the dynamic hosts.

## Requirements
* A suitable system on which to run this, typically, Linux.
* Python3.10+

Just standard Python libraries are used. 

## Installation

1. Clone this respository
2. Change to the cloned directory, typically, cfupdate. 
3. Copy `cfupdate` to a location where it will run from. (optioal) 
4. Make `cfupdate` exeutable.
5. Edit cfupdate.sample.json to cfupdate.json and add your specific information.
6. Copy cfupdate.json to a location where cfupdate wull run. (optional)
7. Install Systemd  units or cronjobs. See [Systemd](#Systemd) or [cron](#cron)

It is perfectly ok to run this from the clined repo directory. 

## Configuraiton

A JSON file is used to configure the tool, in the form of: 

```json
{
  "api_token": "vloudflare-api-key",
  "zones": {
    "domain.com": {
      "zoneid": "cloudflare-zone-id-domain.com",
      "records": {
        "@": {"type": "A", "ttl": 60, "proxied": false},
        "www": {"type": "A", "ttl": 60, "proxied": true},
        "dfrb": {"type": "A", "ttl": 60, "proxied": false}
      }
    },
    "domain.org": {
      "zoneid": "cloudflare-zone-id-domain.org",
      "records": {
        "@": {"type": "A", "ttl": 60,"proxied": false},
        "www": {"type": "A", "ttl": 60,"proxied": true},
        "dfrb": {"type": "A", "ttl": 60,"proxied": false},
        "mx": {"type": "A", "ttl": 60,"proxied": false}
      }
    }
  }
}
```

In order to use this tool you will need, from your Cloudflare Dashboard, your zone id, and your api key. 
Each domain has it own id. This file is organized as zones, with the zone id and records. 
Each record is keyed by the host. Each host as the following attributes:

|Attrinute|Usage                                                                                  |
|---------|---------------------------------------------------------------------------------------|
| type    | Record type, A, AAAA, TXT, CNAME, etc.                                                |
| ttl     | Time-to-live. This is number of seconds record lives before being queried again       |
| proxied | True if proxied, false otherwise                                                      |

## Program Options

|Option   |Usage                                                                                  |
|---------|---------------------------------------------------------------------------------------|
|    -c   |  -c <file> Load config from file                                                      |
|    -l   |  -l <file> Set log file to file. If not set log is to stdout                          |
|    -v   |  -v <level> Set log level to: debug,info,warn,error,fatal. [See Logging](#logging)    |

## Logging 

Logging is performed using Python's logging module. Logging to a file is recommended if running from cron, however, when no log file is set, the logging, without time stamps is done to stdout. This is useful as a systemd service because systemd will send this to the journal.

*It is recommended to set the log level to INFO while ensuring the script runs and there are no glitches.*


## Obtaining the Public IP Address

This tool uses ipify to get the external address with a request to their site. This way the updated
IP address is always current. 

## Systemd
To set up the systemd service edit `cfupdate.service` to update the the path to cfupdate, command line options, and user/group.  Then copy the files to /etc/systemd/system

```shell
sudo cp cfupdate.{timer,service} /etc/systemd/system
sudo systemctl daemon-reload
sudo systemctl enable --now cfupdate.timer.
```

### Service Unit

```shell
[Unit]
Description=Cloudflare Dynamic DNS Update Service

[Service]
Type=oneshot
User=someuser
Group=somegroup
ExecStart=/path/to/cfupdate -c /path/to/cfupdate.json -v info
```

### Timer Unit

```shell
[Unit]
Description=Run Cloudflare Dynamic DNS Update every 5 minutes

[Timer]
OnBootSec=1min
OnCalendar=*:0/5:00
Unit=cfupdate.service

[Install]
WantedBy=timers.target
```

## Cron
Using cron is a little simpler but does not have the same integration with system logging that using systemd does.To add an entry to cron use `crontab -e` and add a line with

```cron
*/5 * * * * /path/to/cfuptate [options]
```
