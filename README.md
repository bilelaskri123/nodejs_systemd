# Run nodejs application with systemd

we will create nodejs application and run it with systemd service

## create node js application
```javascript
const http = require("http");
const hostname = "0.0.0.0"; // listen on all ports
const port = 1337;

http
  .createServer((req, res) => {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("hello world\n");
  })
  .listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
  });
```

we will save this file(for example) as `/opt/nodeserver/server.js`. 
Verify the node server works

```bash
node /opt/nodeserver/server.js
```

# Systemd
## create the service file
create `/etc/systemd/system/nodeserver.service`

```bash
[Unit]
Description=Node.js Example Server
# Requires=After=mysql.service       
# Requires the mysql service to run first

[Service]
ExecStart=/usr/bin/node /opt/nodeserver/server.js
# Required on some systems
# WorkingDirectory=/opt/nodeserver
Restart=always
# Restart service after 10 seconds if node service crashes
RestartSec=10
# output to syslog
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=nodejs-example
#User=<alternate user>
#Group=<alternate group>
Environment=NODE_ENV=production PORT=1337

[Install]
WantedBy=multi-user.target
```

## Enable the service
```bash
systemctl enable nodeserver.service
```

## Start the service
```bash
systemctl start nodeserver.service
```

## Verify it is running
```bash
systemctl status nodeserver.service
```
## Security/testing
Of course this service would run as root, which you probably shouldn't be doing, so you can edit `/etc/systemd/system/nodeserver.service` to run it as a different user depending on your system. If you make any changes to the service file, you will need to do a :
```bash
systemctl daemon-reload
```

before reloading the service
```bash
systemctl restart nodeserver.service
```
I always suggest doing a `systemctl status nodeserver.service` after edits and a restart to ensure the service is running as expected.

```bash
ps -ef | grep server.js # find the current pid
kill 12345 # kill the process by its pid reported above
ps -ef | grep server.js #  notice node process is immediately respawned with a different pid
```

