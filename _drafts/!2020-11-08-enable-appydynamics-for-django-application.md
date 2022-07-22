---
title: Enable AppyDynamics for Django Application
categories: [Tech]
tags: [tips]
date: 2020-11-08
layout: post
---
## Install Agent

Activate the virtual environment you wish to install in and as the owning user run this command (if the application uses the global Python environment, you need to run the install command as root):
```bash
pip install appdynamics gunicorn
```

## Create Configuration File
```ini
[agent]
app = devops
tier = web
node = node 28b3

[controller]
host = marilynxxxx.saas.appdynamics.com
port = 443
ssl = (on)
account = marily_xxx
accesskey = ****

```
Copy and paste above lines and save this file to `/etc/appdynamics.cfg`. This configuration file contains the necessary settings the agent requires to begin connecting to this Controller.

## Instrument the Application

Prepend to your existing application run command the pyagent run command, passing the AppDynamics configuration file. For example, if your current run command looks like this:
```bash
gunicorn -w 8 -b '0.0.0.0:9000' example.app:application
gunicorn -w 2 -b '0.0.0.0:8000' gspdevops.wsgi
```

Replace it with this:
```bash
# usage
pyagent run -c <path_to_appdynamics_config_file> -- gunicorn -w 8 -b '0.0.0.0:9000' example.app:application

# real example
pyagent run -c appd.cfg -- gunicorn -b '0.0.0.0:8080' gspdevops.wsgi
```

