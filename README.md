# chat-app-in-java
chat application written in java
package zappa;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.Socket;
import java.net.UnknownHostException;

import javax.swing.text.html.HTMLDocument.HTMLReader.IsindexAction;

public class Client implements Runnable {
	static Socket connectionSocket = null;
	static BufferedReader ulazniTokOdServera = null;
	static PrintStream izlazniTokKaServeru = null;
	static boolean ende = false;
	static int udpPort;
	static ClientGUI cgi = null;
	Thread t = null;
	
	public Client() {
		t = new Thread(this);
		t.start();
	}
	
	public static void main(String[] args) {
		int port = 8090;
		
		if(args.length>0) {
			port = Integer.parseInt(args[0]);
		}
		
		try {
			connectionSocket = new Socket("localhost", port);
			ulazniTokOdServera = new BufferedReader(new InputStreamReader(connectionSocket.getInputStream()));
			izlazniTokKaServeru = new PrintStream(connectionSocket.getOutputStream());
			
			cgi = new ClientGUI();
			cgi.setVisible(true);
			new Client();
			
			while(!ende) {}
			
			connectionSocket.close();
			
		} catch (UnknownHostException e) {
			System.err.println(e);
		} catch(IOException e) {
			System.err.println(e);
		}
		
	}
	@Override
	public void run() {
		String lajna = "";
		boolean keepGoing = false;
		
		try {
			while((lajna = ulazniTokOdServera.readLine().trim())!=null) {
				if(cgi == null) {
					System.out.println(lajna);
				} else {
					cgi.myAppend(lajna);
				}
				if(lajna.startsWith("** Dovidjenja")) {
					ende = true;
					break;
				}
				if(!keepGoing && Character.isDigit(lajna.charAt(lajna.length()-1)) && Character.isDigit(lajna.charAt(lajna.length()-2))
						&& Character.isDigit(lajna.charAt(lajna.length()-3)) && Character.isDigit(lajna.charAt(lajna.length()-4))
						&& Character.isDigit(lajna.charAt(lajna.length()-5))) {
					keepGoing = true;
					udpPort = Integer.parseInt(lajna.substring(lajna.length()-5));
					new ClientThread(udpPort);
				}
			}
		} catch(IOException e) {
			System.err.println(e);
		}

	}
}

