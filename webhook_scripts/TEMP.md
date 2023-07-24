WEBHOOK_HOST=http://192.168.20.10/alert
tcpdump -i lo -U not udp and not icmp and not port 5007 and not port 5060 and not port 20000 and not dst port 30007 -A -q -l 2>&1 | awk -v host=$WEBHOOK_HOST '/\*#7\*\*31#2\*[0-9][0-9]##/ {system("curl -X GET --data " $1 " " host)}'

tcpdump -i lo -U not udp and not icmp and not port 5007 and not port 5060 and not port 20000 and not dst port 30007 -A -q -l 2>&1 | awk '/\*#7\*\*31#2\*[0-9][0-9]##/ {print "New data: "$1}'
tcpdump -i lo -U not udp and not icmp and not port 5007 and not port 5060 and not port 20000 and not dst port 30007 -A -q -l 2>&1 | awk '/##/ {print "New data: "$1}'

echo *#7**31#2*90## | awk -v host=$WEBHOOK_HOST '/\*#7\*\*31#2\*[0-9][0-9]##/ {print "New data: "$1}'


search patterns
*#7**31#2*10##  - last 2 digits ([0-9][0-9]) changes based on volume (\d\d isn't compatible with awk regex engine)