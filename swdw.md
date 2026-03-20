127.0.0.1;nmap -sC -sV -p1883,8883 127.0.0.1

127.0.0.1; mosquitto_sub -t '#' -d 2>&1