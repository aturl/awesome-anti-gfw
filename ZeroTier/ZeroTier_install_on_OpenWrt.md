```
iptables -I FORWARD -i ztyouqrusr -j ACCEPT
iptables -I FORWARD -o ztyouqrusr -j ACCEPT
iptables -t nat -I POSTROUTING -o ztyouqrusr -j MASQUERADE
iptables -t nat -A POSTROUTING -j MASQUERADE
```
