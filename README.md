# chat-app-in-java
chat application written in java
package zappa;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;

public class ClientThread extends Thread {
	
	static DatagramPacket paket = null;
	static DatagramSocket datagramSocket = null;
	byte[] podaci = null;
	int udpPort;
	
	public ClientThread(int udpPort) {
		this.udpPort = udpPort;
		start();
	}
	
	@Override
	public void run() {
		try {
			datagramSocket = new DatagramSocket(udpPort);
		
			while(!Client.ende) {
				try {
					podaci = new byte[1024];
					paket = new DatagramPacket(podaci, podaci.length);
					datagramSocket.receive(paket);
					if(Client.cgi == null) {
						System.out.println(new String(paket.getData()));
					} else {
						Client.cgi.myAppend(new String(paket.getData()));
					}
					podaci = null;
				} catch(IOException e) {
					continue;
				}
			}
			datagramSocket.close();
		} catch(SocketException e) {
			
		}

	}
	
}
