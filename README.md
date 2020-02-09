# bogons_nftables

Drop bogons in nftables with automatic update and based on fullbogons by Team Cymru

# Install

## Prepare directory

`mkdir -p /root/bogons`

## Script

```
cp bogons_nftables /usr/local/sbin/
chmod +x /usr/local/sbin/bogons_nftables
chown root:root /usr/local/sbin/bogons_nftables
```

## Service

```
cp bogons_nftables.service /etc/systemd/system/
cp bogons_nftables.timer /etc/systemd/system/`
chown root:root /etc/systemd/system/bogons_nftables.service
chown root:root /etc/systemd/system/bogons_nftables.timer
systemctl enable bogons_nftables.service bogons_nftables.timer
systemctl start bogons_nftables.timer
```

## nftables

```
table ip filter {
    set bogons4 {
        type ipv4_addr
            flags interval
            elements = { 0.0.0.0/8 }
    }

    chain FORWARD {
        oifname "eno1" ip daddr @bogons4 drop
    }

    chain OUTPUT {
        ip daddr @bogons4 drop
    }

    chain pre_raw {
        type filter hook prerouting priority -300; policy drop;
        ip saddr @bogons4 drop
            accept
    }
}

table ip6 filter {
    set bogons6 {
        type ipv6_addr
            flags interval
            elements = { ::/8 }
    }

    chain FORWARD {
        ip6 daddr @bogons6 drop
    }

    chain OUTPUT {
        ip6 daddr @bogons6 drop
    }

    chain pre_raw {
        type filter hook prerouting priority -300; policy drop;
        ip6 saddr @bogons6 drop
            accept
    }
}
```
