127.0.0.1;nmap -sC -sV -p1883,8883 127.0.0.1

127.0.0.1; mosquitto_sub -t '#' -d 2>&1

127.0.0.1; python3 -c "import paho.mqtt.client as mqtt,time; c=mqtt.Client(); c.connect('172.18.0.8',1883); 

c.publish('hospital/room1/cardiac','{\"room\":\"room1\",\"temperature\":42.5,\"humidity\":99.9,\"timestamp\":'+str(time.time())+'}'); print('FAKE DATA PUBLISHED on hospital/room1/env'); c.disconnect()"