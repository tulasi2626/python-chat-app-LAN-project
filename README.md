# python-chat-app-LAN-project
**1.server.py - chat server**

import socket
import threading

HOST = '127.0.0.1'
PORT = 12345
clients = []
usernames = []

def broadcast(message, sender=None):
    for client in clients:
        if client != sender:
            client.send(message)

def handle_client(client):
    username = client.recv(1024).decode()
    usernames.append(username)
    welcome = f"{username} joined the chat!\n".encode()
    broadcast(welcome, client)
    log_message(welcome.decode())

    while True:
        try:
            msg = client.recv(1024)
            if msg.decode().startswith("/exit"):
                raise Exception
            message = f"{username}: {msg.decode()}\n"
            broadcast(message.encode(), client)
            log_message(message)
        except:
            index = clients.index(client)
            clients.remove(client)
            left_user = usernames.pop(index)
            client.close()
            left_msg = f"{left_user} left the chat.\n"
            broadcast(left_msg.encode())
            log_message(left_msg)
            break

def receive_connections():
    server.listen()
    print("Server is running...")
    while True:
        client, addr = server.accept()
        clients.append(client)
        print(f"Connected: {addr}")
        thread = threading.Thread(target=handle_client, args=(client,))
        thread.start()

def log_message(message):
    with open("chat_log.txt", "a") as f:
        f.write(message)

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind((HOST, PORT))

receive_connections()

**2.client.py - chat client with GUI**

import socket
import threading
import tkinter as tk
from tkinter import simpledialog, scrolledtext, messagebox

HOST = '127.0.0.1'
PORT = 12345

class Client:
    def _init_(self):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.gui_done = False
        self.running = True

        self.username = simpledialog.askstring("Username", "Enter your name:")

        try:
            self.sock.connect((HOST, PORT))
            self.sock.send(self.username.encode())
        except:
            messagebox.showerror("Connection Error", "Server not available")
            exit(1)

        gui_thread = threading.Thread(target=self.gui_loop)
        recv_thread = threading.Thread(target=self.receive)
        gui_thread.start()
        recv_thread.start()

    def gui_loop(self):
        self.win = tk.Tk()
        self.win.title(f"Chat - {self.username}")

        self.text_area = scrolledtext.ScrolledText(self.win)
        self.text_area.pack(padx=10, pady=10)
        self.text_area.config(state='disabled')

        self.input_area = tk.Entry(self.win)
        self.input_area.pack(fill='x', padx=10, pady=5)
        self.input_area.bind("<Return>", self.write)

        self.win.protocol("WM_DELETE_WINDOW", self.stop)
        self.gui_done = True
        self.win.mainloop()

    def write(self, event=None):
        msg = self.input_area.get()
        if msg == "/exit":
            self.stop()
            return
        self.sock.send(msg.encode())
        self.input_area.delete(0, tk.END)

    def receive(self):
        while self.running:
            try:
                msg = self.sock.recv(1024).decode()
                if self.gui_done:
                    self.text_area.config(state='normal')
                    self.text_area.insert('end', msg)
                    self.text_area.yview('end')
                    self.text_area.config(state='disabled')
            except:
                break

    def stop(self):
        self.running = False
        self.sock.send("/exit".encode())
        self.sock.close()
        self.win.destroy()
        exit(0)

Client()


