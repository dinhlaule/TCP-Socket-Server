package com.neo.connect;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.Socket;
import java.util.Scanner;

public class Client {
	private InetAddress host;
	private int port;

	public Client(InetAddress host, int port) {
		this.host = host;
		this.port = port;
	}

	private void execute() throws IOException {
		Socket client = new Socket(host, port);
		ReadClient read = new ReadClient(client);
		read.start();
		WriteClient write = new WriteClient(client);
		write.start();
	}

	public static void main(String[] args) throws IOException {
		Client client = new Client(InetAddress.getLocalHost(), 9090);
		client.execute();
	}
}

class ReadClient extends Thread {
	private Socket client;

	public ReadClient(Socket client) {
		this.client = client;
	}

	@Override
	public void run() {
		BufferedReader dis = null;
		try {
			dis = new BufferedReader(new InputStreamReader(client.getInputStream()));
			int i;
			while ((i = dis.read()) != -1) {
				System.out.print((char) i);
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				dis.close();
				client.close();
			} catch (IOException ex) {
				System.out.println("Disconnect with Server");
			}
		}
	}

}

class WriteClient extends Thread {
	private Socket client;

	public WriteClient(Socket client) {
		this.client = client;
	}

	@Override
	public void run() {
		PrintWriter dos = null;
		BufferedWriter bw = null;
		Scanner sc = null;
		try {
			dos = new PrintWriter(client.getOutputStream(), true);
			bw = new BufferedWriter(dos);
			sc = new Scanner(System.in);
			while (true) {
				String sms = sc.nextLine();
				if (sms.equals("hello")) {
					InetAddress ip = InetAddress.getLocalHost();
					System.out.println("IP Server: " + ip.getHostAddress().toString());
					System.out.println("Host Name: " + ip.toString());
				} else if (sms.equals("bye")) {
					System.out.println("end");
				}
				bw.write(sms);
				bw.flush();
				bw.close();

			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
