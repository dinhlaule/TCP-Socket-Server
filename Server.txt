package com.neo.connect;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;
import java.util.Scanner;

public class Server {
	private int port;
	public static ArrayList<Socket> listSk;

	public Server(int port) {
		this.port = port;
	}

	private void execute() throws IOException {
		ServerSocket server = new ServerSocket(port);
		System.out.println("Listening...");
		WriteServer write = new WriteServer();
		write.start();
		while (true) {
			Socket socket = server.accept();
			System.out.println("Connected with " + socket);
			Server.listSk.add(socket);
			ReadServer read = new ReadServer(socket);
			read.start();
		}
	}
	
	public static void main(String[] args) throws IOException {
		Server.listSk = new ArrayList<>();
		Server server = new Server(9090);
		server.execute();
	}

}

class ReadServer extends Thread {
	private Socket socket;

	public ReadServer(Socket socket) {
		this.socket = socket;
	}
	
	@Override
	public void run() {
		try {
			BufferedReader dis = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			while (true) {
				String sms = dis.readLine();
				if(sms.equals("hello")) {
					System.out.println("hello");
					InetAddress ip = InetAddress.getLocalHost();
					System.out.println("IP Server: " + ip.getHostAddress().toString());
					System.out.println("Host Name: " + ip.toString());
					continue;
				} else if(sms.equals("list")) {
					System.out.println("list");
					for (int i = 0; i < Server.listSk.size(); i++) {
						System.out.println(Server.listSk.get(i));
					}
					continue;
				} else if(sms.equals("bye")) {
					Server.listSk.remove(socket);
					System.out.println("Disconnected with " + socket);
					dis.close();
					socket.close();
					continue;
				}
				for (Socket item : Server.listSk) {
					if (item.getPort() != socket.getPort()) {
						DataOutputStream dos = new DataOutputStream(item.getOutputStream());
						dos.writeUTF(sms);
					}
				}
				System.out.println(sms);
			}
		} catch (Exception e) {
			try {
				socket.close();
			} catch (IOException e2) {
				System.out.println("Ngắt kết nối Server!");
			}
		}
	}

}

class WriteServer extends Thread {

	@Override
	public void run() {
		PrintWriter dos = null;
		BufferedWriter bw = null;
		Scanner sc = new Scanner(System.in);
		while (true) {
			String sms = sc.nextLine();
			try {
				for (Socket item : Server.listSk) {
					dos = new PrintWriter(item.getOutputStream());
					bw = new BufferedWriter(dos);
					bw.write(sms);
					bw.flush();
				}
				
			} catch (IOException e) {
				e.printStackTrace();
			}
			
		}

	}
}
