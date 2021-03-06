# letsproxy

letsproxy is an easy to use proxy for https data traffic using [Let's Encrypt](https://letsencrypt.org/) certificates.

## Features
* Redirectes requests to http (port 80) to https (port 443)
* Automatically update certificates
* Support websockets
* Proxy target defined in a configuration file

## Give node access to use port 80 and 443
Since the proxy needs to listen to port 80 and port 443 which both are below 1024 the node process needs special privileges to avoid having to run as root. These privileges can be given to the node binary using the following command.
**Please note** that this will give all node processes this privileges.

```bash
# Not that the node path might be a symlink, you must find the real path
sudo setcap 'cap_net_bind_service=+ep' /usr/bin/nodejs
```

## Install letsencrypt cli
https://letsencrypt.org/getting-started/

```bash
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrupt-auto
```

## Create certificates
```bash
letsencrypt-auto certonly --standalone -d example.com -d www.example.com

# /etc/letsencrypt needs to be read and writable as the user running letsproxy
sudo chmod g+rwx /etc/letsencrypt -R
sudo chown root:$USER /etc/letsencrypt -R

# It also seems that you have to symlink the main certificate for all subdomains (Probably letsproxy does something wrong...)
sudo ln -s /etc/letsencrypt/live/example.com /etc/letsencrypt/live/www.example.com
```

## Create configuration
/etc/letsproxy.json

```json
{
  "https": {
    "dir": "/etc/letsencrypt",
    "key": "live/example.com/privkey.pem",
    "cert": "live/example.com/cert.pem"
  },
  "paths" : {
    "example.com": "localhost:8080",
    "www.example.com": "localhost:8080"
  }
}
```

## Run letsproxy
```bash
./bin/letsproxy

# Or if running PM2
pm2 start ./lib/main.js --name LetsProxy
```

## Tips

### Print information about certificate
```bash
openssl x509 -in /etc/letsencrypt/live/example.com/cert.pem -text -noout
```

### Blocking an external port but allow localhost
```bash
iptables -A INPUT ! -s 127.0.0.1 -p tcp -m tcp --dport 8080 -j DROP
```
