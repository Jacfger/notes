# How to Reverse Tunnel to a Server (and probably X11 Forward as well)

To be filled.

# Reverse Tunnel to Server disconnect automatically and Client Wouldn't Reconnect automatically

## Client cannot (or did not) Reconnect to the Server

### Server Side

1. Server's sshd fails to drop the LISTEN port, which needs to be dropped manually after the client disconnect accidentally.
  - ```sudo netstat -tulpn | grep LISTEN``` and search for sshd, kill the process that is using the port ```kill <pid>```. Which would release the port and allow new client to reverse ssh to the server.
  
### Client Side

1. Client wasn't set to kill the ssh if forward fails. (such that service was not restart)
  - Append ```-o ExitOnForwardFailure=yes``` to the ssh command, or add ```ExitOnForwardFailure yes``` to ~/.ssh/config (/etc/ssh/ssh_config should work as well) [1]

2. Use autossh to handle the restarting.
  - ```autossh -M 20000 -f -N your_public_server -R 1234:localhost:22 -C```[2]
  
## Connection is dropped accidentally (probably because of stale connection or timeout) [3]

### Server Side

1. In ```/etc/ssh/sshd_config```

```
  ClientAliveInterval 60
  TCPKeepAlive yes
  ClientAliveCountMax 10000
```

**ClientAliveInterval** The server will wait 60 seconds before sending a null packet to the client to keep the connection alive

**TCPKeepAlive** Is there to ensure that certain firewalls don't drop idle connections.

**ClientAliveCountMax** Server will send alive messages to the client even though it has not received any message back from the client.

### Client Side

1. In ```/etc/ssh/ssh_config```

```
Host *
ServerAliveInterval 100
```

**ServerAliveInterval** The client will send a null packet to the server every 100 seconds to keep the connection alive

**NULL packet** Is sent by the server to the client. The same packet is sent by the client to the server. A TCP NULL packet does not contain any controlling flag like SYN, ACK, FIN etc. because the server does not require a reply from the client. The NULL packet is described here: https://tools.ietf.org/html/rfc6592

# Reference

1. https://superuser.com/questions/352268/can-i-make-ssh-fail-when-a-port-forwarding-fails
2. https://superuser.com/questions/37738/how-to-reliably-keep-an-ssh-tunnel-open
3. https://unix.stackexchange.com/questions/200239/how-can-i-keep-my-ssh-sessions-from-freezing
