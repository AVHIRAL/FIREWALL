# COMMANDE SSH POUR CONTRER PLUSIEURS ATTAQUES SYN, ETC....

netstat -an | grep SYN_RECV

echo "1" > /proc/sys/net/ipv4/tcp_syncookies
echo "1024" > /proc/sys/net/ipv4/tcp_max_syn_backlog
echo "1" > /proc/sys/net/ipv4/conf/all/rp_filter 

Il est possible de paramétrer ses valeurs de façon permanente en modifiant le fichier 
nano /etc/sysctl.conf :

net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_max_syn_backlog = 1024 

sysctl -p /etc/sysctl.conf

IPTABLES protection Syn-attaque

for i in ` netstat -tanpu | grep "SYN_RECV" | awk {'print $5'} | cut -f 1 -d ":" | sort | uniq -c | sort -n | awk {'if ($1 > 3) print $2'}` ; do echo $i; iptables -A INPUT -s $i/24 -j DROP; done

/sbin/iptables -A INPUT -p tcp --syn -m limit --limit 2/s --limit-burst 30 -j ACCEPT
/sbin/iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
/sbin/iptables -A INPUT -p tcp --tcp-flags ALL NONE -m limit --limit 1/h -j ACCEPT
/sbin/iptables -A INPUT -p tcp --tcp-flags ALL ALL -m limit --limit 1/h -j ACCEPT

iptables -I INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -m limit --limit 1/sec -j LOG --log-prefix "iptables input" --log-level 6

iptables -I INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -j DROP

sudo iptables -I INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 10 -j DROP

iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP

iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP

iptables -t mangle -A PREROUTING -p tcp -m conntrack --ctstate NEW -m tcpmss ! --mss 536:65535 -j DROP

iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN FIN,SYN -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,RST FIN,RST -j DROP
