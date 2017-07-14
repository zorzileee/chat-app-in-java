# chat-app-in-java
chat application written in java
package zokiServer;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.io.PrintWriter;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.Socket;
import java.net.SocketException;
import java.util.GregorianCalendar;
import java.util.LinkedList;

public class ServerskaNit implements Runnable {
	
	LinkedList<ServerskaNit> izabrani;
	LinkedList<String> imenaIzabranih;
	BufferedReader ulazniTokOdKlijenta = null;
	PrintStream izlazniTokKaKlijentu = null;
	Thread t = null;
	InetAddress ipClient = null;
	Socket connectionSocket = null;
	int udpPort;
	String ime=null,pol = null,poruka = null;
	DatagramSocket datagramSocket = null;
	byte[] podaciZaKlijenta = null;
	//PrintWriter outFile ;
	
	public ServerskaNit(Socket connectionSocket, int udpPort) {
		this.connectionSocket = connectionSocket;
		//this.listaKlijenata = listaKlijenata;
		this.udpPort = udpPort;
		ipClient = connectionSocket.getInetAddress();
	}
	
	public String getIme() {
		return ime;
	}
	public String getPoruka() {
		return poruka;
	}

	public String getPol() {
		return pol;
	}
	public void connect() {
		try {
			
			ulazniTokOdKlijenta = new BufferedReader(new InputStreamReader(connectionSocket.getInputStream()));
			izlazniTokKaKlijentu = new PrintStream(connectionSocket.getOutputStream());
			//outFile = new PrintWriter(new BufferedWriter(new FileWriter("zajednicki.txt")));
			boolean ende = false;
			
			while(!ende) {
				izlazniTokKaKlijentu.println("*--> Unesite ime : ");
				ime = ulazniTokOdKlijenta.readLine().trim();
				if(!Server.imenaKlijenata.contains(ime)) {
					addToListOfNames(ime);
					ende = true;
				} else {
					izlazniTokKaKlijentu.println("** Vec postoji osoba, pokusajte ponovo : ");
				}
			}
			
			izlazniTokKaKlijentu.println("*--> Unesite pol(m/z) : ");
			pol = ulazniTokOdKlijenta.readLine().trim();
			boolean greska = !pol.startsWith("m") && !pol.startsWith("z");
			
			if(greska) {
				izlazniTokKaKlijentu.println("*--> Imate jos jednu sansu, nakon toga kraj(m/z) : ");
				pol = ulazniTokOdKlijenta.readLine().trim();
				
				if(greska) {
					izlazniTokKaKlijentu.println("** Ende.");
					connectionSocket.close();
					datagramSocket.close();
					removeFromListOfNames(ime);;
					removeFromList(this);
					if(this.t.isAlive())
						this.t.interrupt();
				}
			} else {
				Server.welcomeSignals(this);
			}
		} catch(IOException e) {
			//e.printStackTrace();
		}
	}
	public static void addToList(ServerskaNit noviKlijent) {
		synchronized(Server.listaKlijenata) {
			Server.listaKlijenata.add(noviKlijent);
		}
	}
	public static void addToListOfNames(String novoIme) {
		synchronized(Server.imenaKlijenata) {
			Server.imenaKlijenata.add(novoIme);
		}
	}
	public static void removeFromList(ServerskaNit klijent) {
		synchronized(Server.listaKlijenata) {
			Server.listaKlijenata.remove(klijent);
		}
	}
	public static void removeFromListOfNames(String name) {
		synchronized(Server.imenaKlijenata) {
			Server.imenaKlijenata.remove(name);
		}
	}
	public void disconnect() {
		try {
		for(ServerskaNit s : Server.listaKlijenata) {
			if(s!=null && s.getIme()!=null && s.getPol()!=null && !(s.getPol().equalsIgnoreCase(pol))) {
				if(pol.startsWith("m")) {
					s.izlazniTokKaKlijentu.println("** Korisnik "+ime+" napusta chat sobu.");
				} else {
					s.izlazniTokKaKlijentu.println("** Korisnica "+ime+" napusta chat sobu.");
				}
			}
		}
		izlazniTokKaKlijentu.println("** Dovidjenja, "+ime+".");
		connectionSocket.close();
		datagramSocket.close();
	//	outFile.close();
		removeFromListOfNames(ime);
		removeFromList(this);
		if(this.t.isAlive())
			this.t.interrupt();
		
		} catch(IOException e) {
			e.printStackTrace();
		}
	}
	public boolean emptyRoom() {
		if(Server.listaKlijenata.isEmpty())
			return true;
		int i = 0;
		for(ServerskaNit s : Server.listaKlijenata) {
			if(s!=null && s.getIme()!=null && s.getPol()!=null && !(s.getPol().equalsIgnoreCase(pol))) {
				i++;
			}
		}
		return (i == 0) ? true : false;
	}
	public void sendClientList() {
		byte[] podaci1 = new byte[1024];
		byte[] podaci2 = new byte[1024];
		String lajna = null;
		try {
			while(true) {
				if(emptyRoom()) {
					podaci1 = "** Soba je trenutno prazna,".getBytes();
					DatagramPacket paket1 = new DatagramPacket(podaci1, podaci1.length, ipClient, udpPort);
					datagramSocket.send(paket1);
					podaci2 = "** Za ponovnu proveru dostupnih korisnika, unesi !pp, za izlazak !quit".getBytes();
					DatagramPacket paket2 = new DatagramPacket(podaci2, podaci2.length, ipClient, udpPort);
					datagramSocket.send(paket2);
					
					lajna = ulazniTokOdKlijenta.readLine().trim();
					if(lajna.startsWith("!quit")) {
						disconnect();
					} else continue;
				} else {
					String lista = (pol.startsWith("m")) ? "** Online korisnice: " : "** Online korisnici: ";
					for(ServerskaNit s : Server.listaKlijenata) {
						if(s!=null && s.getIme()!=null && s.getPol()!=null && !(s.getPol().equalsIgnoreCase(pol))) {
							lista += s.getIme()+", ";
						}
					}
					lista += "/**";
					podaciZaKlijenta = lista.getBytes();
					DatagramPacket paketZaKlijenta = new DatagramPacket(podaciZaKlijenta, podaciZaKlijenta.length, ipClient, udpPort);
					datagramSocket.send(paketZaKlijenta);
					return;
				}
			}
		} catch(IOException e) {
			
		}
	}
	public void chooseClientsToCommunicateWith() {
		izabrani = new LinkedList<>();
		imenaIzabranih = new LinkedList<>();
		try {
			izlazniTokKaKlijentu.println("*--> Unesite imena korisnika kojima zelite da posaljete poruku, nakon unosa svakog imena stavite zarez : ");
			String[] za = (ulazniTokOdKlijenta.readLine().trim()).split(",");
			for(String s : za) {
				if(s!=null && Server.imenaKlijenata.contains(s))
					for(ServerskaNit nit : Server.listaKlijenata) {
						if(nit!=null && nit.getIme()!=null && nit.getPol()!=null && nit.getIme().equalsIgnoreCase(s)) {
							izabrani.add(nit);
						}
					}
			}
			
		} catch(IOException e) {
			
		}
		
	}
	public void communicate() {
		try {

			//outFile = new PrintWriter(new BufferedWriter(new FileWriter(Server.file)));
			while(true) {
				poruka = ulazniTokOdKlijenta.readLine().trim();
				if(poruka.startsWith("!quit")) {
					disconnect();
					return;
				} else if(poruka.startsWith("!pp")) {
					sendClientList();
				} else {
					sendClientList();
					chooseClientsToCommunicateWith();
					try {
						String online = "";
						String offline = "";
						for(ServerskaNit s : izabrani) {
							if(Server.listaKlijenata.contains(s)) {
								online += s.getIme() + ", ";
								s.izlazniTokKaKlijentu.println(ime+" salje: "+poruka);
							} else {
								offline += s.getIme() + ", ";
								izabrani.remove(s);
							}
						}
						if(!online.equalsIgnoreCase("")) {
							izlazniTokKaKlijentu.println("** Uspesno slanje poruke sledecim korisnicima : "+online);
						}
						if(!offline.equalsIgnoreCase("")) {
							izlazniTokKaKlijentu.println("** Neuspesno slanje poruke sledecim korisnicima : "+offline);
						}
						/*outFile.println(ime+" salje: "+poruka+", vreme: "+new GregorianCalendar().getTime().toString());
						outFile.println("Poruka nije uspesno poslata: "+offline);
						outFile.println("Poruka je uspesno poslata: "+online);*/
						Server.writeInFile1(getIme(),getPol());
						Server.writeInFile2(online, offline);
					} catch(Exception e) {
						izlazniTokKaKlijentu.println("** Greska pri slanju poruke! ");
					}
				}
			}
		} catch(IOException e) {
			e.printStackTrace();
		}
	}
	public void formUDP() {
		try {
			datagramSocket = new DatagramSocket();
			podaciZaKlijenta = new byte[1024];
		} catch (SocketException e) {
			// TODO Auto-generated catch block
		}
	}
	@Override
	public void run() {
		
		formUDP();
		connect();
		communicate();

	}
	public void myStart() {
		if(t == null) {
			t = new Thread(this);
			t.start();
		}
	}

}
