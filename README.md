# P2P File Transfer


This project was completed for the Distributed Systems course at UFABC - Universidade Federal do ABC. The original project was implemented in Java, but I decided to redo the exercise in Rust (without necessarily following all the restrictions of the original project).

## How to run

### Server:

`cargo run -p server`

### Peer:
`cargo run -p server`

___

My original submission for the course can be found at this link: [projeto-sistemas-distribuidos](https://github.com/Gustavo2358/projeto-sistemas-distribuidos).

Below is the task specification:


## Programming Project - Napster

### 1. System Definition

Create a P2P system that allows the transfer of large video files (over 1 GB) between peers, intermediated by a centralized server, using TCP and UDP as transport layer protocols.

The system will function similarly (but on a much smaller scale) to the Napster system, one of the first P2P systems created by Shawn Fanning at the age of 18.

### 2. Initial Recommendation

If you have never programmed with TCP or UDP, I recommend watching the video at the link [here](https://www.youtube.com/watch?v=nysfXweTI7o) and implementing the examples shown. Just watching the video will not be useful when you have to implement more complex functionalities.

### 3. System Overview

The system will consist of 1 server (with known IP and port) and many peers. The peer acts both as an information provider (in this case, files) and as a receiver of them.

Initially, the server will be available to receive requests from peers. When PeerX enters the system, it must communicate its information to the server. The server will receive the information and store it for future queries. When PeerY wants to download a video, it should send a request to the server with the file name. The server will search for the name and respond to PeerY with a list of peers that have it. PeerY will receive the list from the server and choose one of the peers from the list (let's assume the chosen one is PeerZ). Then, PeerY will request the file from PeerZ, who can either accept the request, sending the file, or reject the request. Finally, when PeerY downloads the file into a folder, the person can go to the folder and view it using an external player, like VLC.

### 4. Server Functionalities

- Receives and responds simultaneously (with threads) to requests from peers.
- **JOIN Request**: From a peer that wants to join the network. The request must contain the minimum information of the peer (e.g., file names it has), which should be stored in some data structure on the server. The server's response sent to the peer will be the string `JOIN_OK`.
- **LEAVE Request**: From a peer that wants to leave the network. The information of this peer stored on the server must be deleted. The server's response sent to the peer will be the string `LEAVE_OK`.
- **SEARCH Request**: From a peer that wants to download a specific file. The request should only contain the file name with its extension (e.g., the string `Aula.mp4`). The server's response sent to the peer will be an empty list or a list with the information of the peers that have the file.
- **UPDATE Request**: From a peer that downloaded a file. The request should contain the name of the downloaded file, which will be inserted into the data structure that holds the peer's information. The server's response sent to the peer will be the string `UPDATE_OK`.
- **ALIVE Request**: Sent every 30 seconds by the server to the peers to check if they are alive or if they left the network abruptly (i.e., without notifying with LEAVE). The server will receive the string `ALIVE_OK` if the peer is alive. If the server does not receive a response from a specific peer, it should delete this peer's information from the data structure.
- **Server Initialization**: The server must initially capture the IP. The IP address to be inserted will be 127.0.0.1. Assume this IP when the Peer wants to communicate with the server. The default port (which will allow peers to connect to it) will be 10098. The capture will be done via the keyboard.

### 5. Peer Functionalities (assume the viewpoint of PeerX)

- Receives and responds simultaneously (with threads) to requests from the server and other peers.
- Sends a JOIN request to the server via UDP. Must wait for JOIN_OK.
- Sends a LEAVE request to the server via UDP. Must wait for LEAVE_OK.
- Sends an UPDATE request to the server via UDP. Must wait for UPDATE_OK.
- Sends an ALIVE_OK response to the server via UDP.
- Sends a SEARCH request to the server via UDP. A list will return as a response (empty or with information).
- For the above UDP sends, if it does not receive a response, it should resend the request, taking care of possible duplications (e.g., server response delays).
- Sends a DOWNLOAD request to another peer via TCP. See below for possible responses.
    - **DOWNLOAD Request**: From another peer (PeerY), requesting a specific file. PeerX must check if it has the file and, randomly, accept or reject the request. If the request is accepted, it will send the file to PeerY via TCP. If rejected, it will send the string `DOWNLOAD_DENIED` to PeerY via TCP.
- **File Reception**: The file should be stored in a specific folder of the peer.
- **DOWNLOAD_DENIED Request**: From the peer that rejected the DOWNLOAD request. PeerX should make the same request to another peer on the list obtained in the SEARCH. If there are none, make the request (after a certain period of time) to the same peer that rejected it.
- **Peer Initialization**: The peer must initially capture the IP, port, and folder where its files are (and will be) stored. The capture will be done via the keyboard. Each peer will have its own folder. For example, if there are 3 peers, there will be 3 different folders. See JOIN in the ‘Interactive Menu’ item in section 6.

### 6. Console Messages (prints)

On the peer's console, the following information should be presented “exactly” (no more, no less):

- When receiving JOIN_OK, print “I am peer [IP]:[port] with files [only file names]”. Replace the information in parentheses with the actual ones. For example: “I am peer 127.0.0.1:8776 with files aula1.mp4 aula2.mp4”.
- When receiving the SEARCH response, print “Peers with the requested file: [IP:port of each peer in the list]”.
- When receiving the file, print “File [only file name] successfully downloaded in folder [folder name]”.
- When receiving DOWNLOAD_DENIED, print “Peer [IP]:[port] denied the download, now requesting from peer [IP]:[port]”.

Interactive menu (via console) allowing only the functions JOIN, SEARCH, DOWNLOAD, and LEAVE to be chosen.

- For JOIN, it must capture only the IP, port, and folder where the files are (e.g., c:\temp\peer1\, c:\temp\peer2\, etc.). From the folder, your code will look for the file names to be sent to the server.
- For SEARCH, it must capture only the file name with its extension (e.g., aula.mp4). The search will be exactly for this name. Note that it should not capture the folder.
- For DOWNLOAD, it must capture only the IP and port of the peer where the file to be downloaded is. Note that it should not capture the folder.

On the server's console, the following information should be presented “exactly” (no more, no less):

- When receiving JOIN, print “Peer [IP]:[port] added with files [only file names]”.
- When receiving SEARCH, print “Peer [IP]:[port] requested file [only file name]”.
- If it does not receive ALIVE_OK, print “Peer [IP]:[port] dead. Removing its files [only file names]”.

### 7. Test Performed by the Professor

The professor will compile the code using the `javac` of JDK 1.8.

After the compilation, the professor will open 4 consoles (on Windows, it would be CMD.EXE, also known as prompt). One will be the server and the other 3 will be peers. From the consoles, the professor will perform the system functionality tests. Examples of the consoles can be seen at the link [here](https://www.youtube.com/watch?v=FOwKxw9VYqI).

It should be noted that:

- Your code should not be limited to 3 peers, supporting more than 3.
- Initially, the professor will start the server. Then, the peers will be started.
- You will not need 4 computers to perform the activity. Just open the 4 consoles.

### 8. Source Code

- You should create only the classes Server, Peer, and Message. The last one should be used mandatorily for sending and receiving information (in requests and responses). If you send new classes, 3 points will be deducted from the final grade.
- The only exception is the creation of classes for Threads, but they should be created within the Peer or Server class (e.g., nested classes).
- The source code should clearly present (using comments) the code sections that perform the functionalities mentioned in Sections 4 and 5.
- The source code should be compiled and executed by a JDK 1.8 (also known as JDK 8). You can use all the existing classes and packages in it. The link to them is [here](https://docs.oracle.com/javase/8/docs/api/).
- The use of libraries that perform part of the requested functionalities will not be accepted. If you have doubts about a specific one, ask the professor.

### 9. Final Submission

The submission is individual and will consist of: (a) a report [YourRA].pdf; (b) the source code of the program with the folders and the respective .java files (c) the link to a video, within the report, showing the system working. The submission will be through Moodle, not accepted by other means.

- If you want to use another programming language, you must send an email to the professor by 07/05 and wait for the professor's acceptance. After this date, the submission must be in Java.
- If you used a library allowed by the professor, send it as well.

The report should have the following sections:

- Name and RA of the participant
- Link to the video showing the system working. The video should be a maximum of 3 minutes, showing the compilation, the code running on the four consoles, and the playback of the transferred file. Do not send the video, just send the video link, which can be made available on YouTube or another similar site (like Vimeo). Remember to give permission to view it.
- For each server functionality, a brief “high-level” explanation of how the request handling was implemented. The explanation MUST mention the lines of the source code that refer to it.
- For each peer functionality, a brief “high-level” explanation of how the request handling was implemented. The explanation MUST mention the lines of the source code that refer to it.
- Explanation of the use of threads. Separate the explanation of the server and peer, mentioning the lines of the source code that refer to it.
- Explanation of how you implemented the transfer of large files.
- Links to the places where you based your code (if applicable). I prefer that you insert the places rather than find something similar on the Internet.

### 10. Important Evaluation Notes

The following issues will deduct from the grade:

- Source code does not compile or uses external libraries without the professor's approval (grade zero).
- Does not transfer files larger than 1GB (minus 2 points).
- Did not use Threads both in Peer and Server, i.e., both (minus 2 points).
- It is not possible to play the transferred file with an external player (minus 1 point).
- Did not submit the report (minus 2 points).
- Did not submit the video link, the link is not available, or the video does not show the system working and playback (minus 2 points).
- Sent other source code files in addition to those mentioned in Section 8, such as .class or other classes (minus 2 points).
- Source code without comments referencing the functionalities of sections 4 and 5 (minus 2 points).

### 11. Recommended Links

- To download JDK 8: [link](https://www.oracle.com/br/java/technologies/javase/javase-jdk8-downloads.html)
- Video tutorial on UDP, TCP, and Threads programming: [link](https://www.youtube.com/watch?v=nysfXweTI7o)
- Information on UDP programming can be found at:
  - [Baeldung](https://www.baeldung.com/udp-in-java)
  - [GeeksforGeeks](https://www.geeksforgeeks.org/working-udp-datagramsockets-java/)
  - [Oracle](https://docs.oracle.com/javase/tutorial/networking/datagrams/index.html)
- Information on TCP programming can be found at:
  - [Baeldung](https://www.baeldung.com/a-guide-to-java-sockets)
- Information on Threads, which allow the server or peer to receive and send information simultaneously:
  - [Baeldung](https://www.baeldung.com/a-guide-to-java-sockets) (Section 6: TCP with multiple clients)
  - [TutorialsPoint](https://www.tutorialspoint.com/java/java_multithreading.htm)

### 12. Ethics

Cheating, fraud, or plagiarism will result in a zero grade for all involved in all assessments and programming exercises of the discipline.

### Frequently Asked Questions (FAQ)

1. **Where do I start?**

As mentioned in Section 2, I recommend watching and implementing the examples shown in the video. With these examples "implemented" and "tested", you will have a working client and server (concurrent) transmitting messages via UDP and TCP. This is the foundation for the Napster project.

2. **I have implemented the UDP and TCP examples from the video and they are working. What now?**

In computing, we have the phrase "divide and conquer" and we can use this concept here. I recommend first implementing the JOIN and see that it is working. Only then will you implement other functionalities, such as LEAVE and SEARCH and concurrency.

For example, let's think only about JOIN from the Peer side. First, implement a method that reads the information from the keyboard. Second, implement a method that reads the information from the folder obtained in the previous step. Third, implement a method that sends this information as a String to the server. Now, let's think only about JOIN from the Server side. First, implement a method that receives the String from the socket and obtains the information. Second, create a structure that stores this information (for example, a Hashtable? or a List?). Third, send the response to the Peer. Now, back to the Peer. Fourth, create a method to receive JOIN_OK. Fifth, create a while loop to perform another operation (note that now you can perform others, such as SEARCH or LEAVE). I recommend doing LEAVE after JOIN, which will allow you to perform tests.

3. **How do I send the Message object instead of the String?**

There are a few ways. You can serialize the object (not recommended) or use the JSON format (recommended). JSON allows transforming a Java object to a String and vice versa. If you want to use JSON, it should be through the [Gson library](https://github.com/google/gson). An interesting link: [Jenkov](http://tutorials.jenkov.com/java-json/gson.html) (only up to "Generating JSON From Java Objects").

4. **Can I use a library (like gson from question 3) to send the 1GB file?**

As mentioned in the document, you cannot use libraries that implement the requested functionalities. On the other hand, there are hundreds of web pages showing how to transmit a large file.

5. **After sending the file from Peer A to Peer B, does Peer A send any more messages?**

No. Peer A only sends the file and Peer B receives it. There is no need to send any other messages after this transmission. Remember that DOWNLOAD_DENIED corresponds to the case where Peer A does not send the file to Peer B.

6. **I have the structure on the server and created the Threads that attend the Peers. I am having concurrency issues when updating the structure. How do I solve this?**

There are several ways, such as using locks, semaphores, synchronized, etc. The simplest is to use a structure that already handles/implements concurrency. For this, Java 1.8 has the concurrent package. For example, you can use ConcurrentHashMap or ArrayBlockingQueue. An interesting link: [Baeldung](https://www.baeldung.com/java-concurrent-map)
