#openvpn-ddns

Maintain dns records for connecting openvpn clients

# Features

- generate PTR/AAAA/A records using `common_name` of the client certifcate
  and both internal or external ip addresses using `nsupdate`
- support for private key
- supports multiple forward and reverse zones and
  pick the first matching ip address or `common_name`

# Usage

The following text assumes, that an openvpn server with client certificates,
as described in the [openvpn documentation](https://openvpn.net/index.php/open-source/documentation/miscellaneous/77-rsa-key-management.html)
and a nameserver which supports update via nsupdate such as [bind](https://www.isc.org/downloads/bind/).

1. Install Ruby and nsupdate

2. Clone Project

```
$ cd /etc/openvpn
$ git clone https://github.com/Mic92/openvpn-ddns.git ddns
```

3. Edit Configuration

```
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/openvpn-ddns.json
```

In case you have multiple openvpn server you can also create a configuration per
profile:

```
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/server1.openvpn-ddns.json
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/server2.openvpn-ddns.json
```

where `server1` or `server2` is the name of the openvpn configuration files without the
`.config/.ovpn` extension.

the configuration takes the following following keys:

- **name_server** (string, required): hostname or ip address to nameserver                                                        |
- **nsupdate_executable** (string, optional): path or name of nsupdate (Defaults to "nsupdate")                                           |
- **private_key** (string, optional): If set, this will be used by nsupdate to authenticate against nameserver, use the format `algorithm:keyname key`,
 for example `hmac-sha512:ddns-key NTc1ODVmNDk5NzgwMDgyODQ2ZTAzMGNlZmI0YTkwN2M5ZTg1MzNiN2UxMWQyNjZhNjg2YWQ1MDc4Y2NlZjU0Mw==`,
 where `keyname` is the name used in nameserver configuration and `algorithm` the used TSIG key algorithm
- **reverse_zones** (array, optional): list of reverse zones, which are matched against public/private openvpn client ip.
 If the ip is contained in one of the provided reverse zones, the PTR records will be updated using the common_name as value.
- **private_zones** (array, optional) list of zones, which are matched against the common_name field of the client certificate.
 If one zone is a suffix of the common_name, a A or AAAA records are updated using the internal ip address as value.
- **private_search_domain** (string, optional) if set and non of the private zones matched, openvpn-ddns will fallback to this domain.
 The record will be build from the host part of `common_name`.
- **public_zones** (array, optional) list of zones, which are matched against the common_name field of the client certificate.
 If one zone is a suffix of the common_name, a A or AAAA records are updated using the public ip address as value.
- **public_search_domain** (string, optional) if set and non of the public zones matched, openvpn-ddns will fallback to this domain.
 The record will be build from the host part of `common_name`.

A dnssec-key can be obtained like this:

```
$ ddns-confgen -q -a hmac-sha512 -k openvpn
key "openvpn" {
        algorithm hmac-sha512;
        secret "NTc1ODVmNDk5NzgwMDgyODQ2ZTAzMGNlZmI0YTkwN2M5ZTg1MzNiN2UxMWQyNjZhNjg2YWQ1MDc4Y2NlZjU0Mw==";
};
```

4. Run openvpn

Add the following lines to you openvpn server configuration.

```
learn-address /etc/openvpn/ddns/openvpn-ddns
script-security 2
```

At the moment only `tun` mode of openvpn is supported.
