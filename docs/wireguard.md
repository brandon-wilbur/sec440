# WireGuard Setup on Ubuntu 20

WireGuard is an easy-to-use VPN software that is rising in popularity. The following guide shows how to setup a WireGuard tunnel on a client and a server running Ubuntu 20.

## Initial Server Setup

From a root shell on the server, install WireGuard:

```
apt update -y
apt install wireguard -y
```

Now, create a keypair:

```
umask 077; wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
ip link add wg0 type wireguard
ip addr add 10.0.0.1/24 dev wg0
wg set wg0 private-key /etc/wireguard/privatekey
wg set wg0 listen-port 51820
ip link set wg0 up
```

Take note of the server's public key:

```
cat /etc/wireguard/publickey
```

## Initial Client Setup

From a root shell:

```
umask 077; wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
ip link add wg0 type wireguard
ip addr add 10.0.0.2/24 dev wg0 # This will be the address to use on the remote network
wg set wg0 private-key /etc/wireguard/privatekey
ip link set wg0 up
```

Take note of the client's public key:

```
cat /etc/wireguard/publickey
```

## Server Setup

Now, configure the server to allow this client to connect:

```
wg set wg0 peer <CLIENT-PUBLIC-KEY-HERE> allowed-ips 10.0.0.48/28
```

Subsitute in the client's public key obtained previously in this command. This `allowed-ips` range will only allow 14 addressable clients to connect; this can be adjusted accordingly.

Write this finished configuration to disk:

```
wg showconf wg0 | tee /etc/wireguard/wg0.conf
```

Additionally, you can specify on the server for Wireguard to start on boot:

```
systemctl enable wg-quick@wg0
```

## Client Setup

Now, configure the client to connect to the proper server:

```
wg set wg0 peer <SERVER-PUBLIC-KEY-HERE> allowed-ips 10.0.0.0/24 endpoint <SERVER-PUBLIC-IP>:<SERVER-PUBLIC-PORT>
```

Substitute the server's public key obtained previously in this command. Additionally, make sure that the server's public IP or hostname is used here so the client can reach the server.

Write this finished configuration to disk:

```
wg showconf wg0 | tee /etc/wireguard/wg0.conf
```

### Connecting

On both the client and the server, run `wg` to get an output of the current state of the service.

To bring the connection up:

```
wg-quick up wg0
```

To stop the connection:

```
wg-quick down wg0
```

### Additional Notes

These finished configuration files can be brought over to other hosts, however the generated keys will need to travel with the configuration file. The best practice is to generate a set of asymmetric keys on each client and then adding each client's public key to the server configuration. Specific addresses can be given with an `allowed-ips` directive ending with a `/32` CIDR notation on the server side. 

Split tunneling will happen by default in this configuration automatically via `allowed-ips`. On the client end, all traffic can be tunneled with `allowed-ips 0.0.0.0/0`, or left so only addresses within the given range are tunneled through WireGuard.