# chat-app-in-java
chat application written in java
package zokiServer;

import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;
import java.util.GregorianCalendar;
import java.util.LinkedList;

public class Server {	
	
	static ArrayList<ServerskaNit> listaKlijenata = new ArrayList<>();
	static ArrayList<String> imenaKlijenata = new ArrayList<>();
	static int udpPort = 20000;
	//static File file = new File("C:\\Users\\Zoran\\workspace\\zokiServer\\zajednicki.txt");
	static File file;
	//static final String PORTFILE = "C:/Users/Zoran/workspace/zokiServer/zajednicki.txt";
	static PrintWriter outFile ;
	
	public static void main(String[] args) {
		int port = 8090;
		
		if(args.length>0) {
			port = Integer.parseInt(args[0]);
		}
		
		ServerSocket vrataDobrodoslice = null;
		
		try {
			file = new File("C:\\Users\\Zoran\\workspace\\zokiServer\\zajednicki.txt");
			//outFile = new PrintWriter(new BufferedWriter(new FileWriter("zajednicki.txt")));
			vrataDobrodoslice = new ServerSocket(port);
			while(true) {
				Socket connectionSocket = vrataDobrodoslice.accept();
				ServerskaNit noviKlijent = new ServerskaNit(connectionSocket, udpPort);
				udpPort++;
				listaKlijenata.add(noviKlijent);
				noviKlijent.myStart();
			}
		} catch(IOException e) {
			System.err.println("Greska!"+e);
		}

	}
	public synchronized static void welcomeSignals(ServerskaNit s) {
		if(s.getPol().startsWith("m")) {
			s.izlazniTokKaKlijentu.println("** Dobrodosao, "+s.getIme()+". Za izlazak unesite !quit, mozete poceti sa unosom poruka. Tvoj broj udp porta: "+s.udpPort);
		} else {
			s.izlazniTokKaKlijentu.println("** Dobrodosla, "+s.getIme()+". Za izlazak unesite !quit, mozete poceti sa unosom poruka. Tvoj broj udp porta: "+s.udpPort);
		}
		for(ServerskaNit ser : Server.listaKlijenata) {
			if(ser!=null && ser.getIme()!=null && ser.getPol()!=null && !(ser.getPol().equalsIgnoreCase(s.getPol()))) {
				if(s.getPol().startsWith("m")) {
					ser.izlazniTokKaKlijentu.println("** Novi korisnik "+s.getIme()+" je usao u chat sobu.");
				} else {
					ser.izlazniTokKaKlijentu.println("** Nova korisnica "+s.getIme()+" je usla u chat sobu.");
				}
			}
		}
	}
	public synchronized static void writeInFile1(String s1, String s2) {
		//try {
			try {
				outFile = new PrintWriter(new BufferedWriter(new FileWriter(file)));
				outFile.println(s1+" salje: "+s2+", vreme: "+((new GregorianCalendar()).getTime()).toString());
				outFile.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			//outFile.write('\n');
			//outFile.close();
		/*} catch (IOException e) {
			// TODO Auto-generated catch block
		}*/
		
	}
	public synchronized static void writeInFile2(String s1, String s2) {
		//try {
			try {
				outFile = new PrintWriter(new BufferedWriter(new FileWriter(file)));
				outFile.println("Poruka je uspesno poslata: "+s1);
				//outFile.write('\n');
				outFile.println("Poruka nije uspesno poslata: "+s2);
				outFile.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			//outFile.write('\n');
			//outFile.close();
		/*} catch (IOException e) {
			// TODO Auto-generated catch block
		}*/
		
	}


}
