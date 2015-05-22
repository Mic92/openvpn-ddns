#openvpn-ddns

Maintain dns records for connecting openvpn clients

# Features

- generate PTR/AAAA/A records from `common_name` of the client cert
  and internal or external ip addresses using `nsupdate`
- support for private key
- supports multiple forward and reverse zones and
  pick the first matching ip address or `common_name`

# Usage

1. Install Ruby

2. Clone Project

```
$ cd /etc/openvpn
$ git clone git.github.com/Mic92/openvpn-ddns ddns
```

3. Edit Configuration

```
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/openvpn-ddns.json
```

In case you have multiple openvpn server you can also create a configuration per
profile

```
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/server1.openvpn-ddns.json
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/server2.openvpn-ddns.json
```

where `server1` or `server2` is the name of the openvpn configuration file without the
`.config/.ovpn` extension.

4. Run openvpn

Either

```
$ openvpn --script-security 2 --learn-address /etc/openvpn/ddns/openvpn-ddns --conf server.conf
```

or

```
$ echo "learn-address /etc/openvpn/ddns/openvpn-ddns"  >> server.conf
$ echo "script-security 2"  >> server.conf
$ openvpn --conf server.conf
```

At the moment only `tun` mode of openvpn is supported.
