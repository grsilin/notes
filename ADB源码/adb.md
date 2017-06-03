# Android Debug Bridge

Android Debug Bridge (adb) is a versatile command-line tool that lets you communicate with a device (an emulator or a connected Android device). 

 It is a client-server program that includes three components:

- **A client**, which sends commands. The client runs on your development machine. You can invoke a client from a command-line terminal by issuing an adb command.
- **A daemon (adbd)**, which runs commands on a device. The daemon runs as a background process on each device.
- **A server**, which manages communication between the client and the daemon. The server runs as a background process on your development machine.

adb client first checks whether there is an adb server progress already running. If there isn't,it starts the server progress.When the server starts,it binds to local TCP port 5037 and listens for commands sent from adb clients - all adb clients use the port 5037  to communicate with the adb server.