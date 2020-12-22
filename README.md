# jenkins-docker-network

Lets you run Jenkins and Agents locally with docker-compose.
Both agents have maven 3.6.3, scala 2.12.12, and sbt 1.4.2 installed.
One agent has openjdk-8 and the other openjdk-11.
The agents can also use docker themselves, via a map to the host docker socket.

Initially you will need to connect to the master on localhost:8888 and go through the setup. You need to configure the first user with username and password 'Jenkins'. Then bring things down and back up so the agents can then connect.

Part of the setup requires you to get the initial login token from inside the master container at the location specified on the setup page.

The host command 'sudo chmod 666 /var/run/docker.sock' will provide the required access to the host docker socket. You will need to do this each time you restart things, but is only required on if you want the agents to run thinghs that use docker themselves.
