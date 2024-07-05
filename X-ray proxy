import socket
import threading
import ssl

def handle_client(client_socket):
    try:
        # Receive the initial request from the client
        request = client_socket.recv(4096).decode('utf-8')
        
        # Parse the request to get the target host and port
        first_line = request.split('\n')[0]
        url = first_line.split(' ')[1]
        http_pos = url.find("://")
        if http_pos == -1:
            temp = url
        else:
            temp = url[(http_pos + 3):]
        
        port_pos = temp.find(":")
        host_pos = temp.find("/")
        if host_pos == -1:
            host_pos = len(temp)
        
        if port_pos == -1 or host_pos < port_pos:
            port = 80
            host = temp[:host_pos]
        else:
            port = int(temp[(port_pos + 1):host_pos])
            host = temp[:port_pos]
        
        # Connect to the target server
        server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server_socket.connect((host, port))
        
        # If it's an HTTPS connection, wrap the socket with SSL
        if port == 443:
            server_socket = ssl.wrap_socket(server_socket)
        
        # Send the request to the server
        server_socket.send(request.encode())
        
        # Set up bidirectional communication
        def server_to_client():
            while True:
                data = server_socket.recv(4096)
                if len(data) > 0:
                    client_socket.send(data)
                else:
                    break
        
        def client_to_server():
            while True:
                data = client_socket.recv(4096)
                if len(data) > 0:
                    server_socket.send(data)
                else:
                    break
        
        t1 = threading.Thread(target=server_to_client)
        t2 = threading.Thread(target=client_to_server)
        
        t1.start()
        t2.start()
        
    except Exception as e:
        print(f"Error: {e}")
    finally:
        client_socket.close()
        server_socket.close()

def main():
    proxy_host = '127.0.0.1'
    proxy_port = 8080
    
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((proxy_host, proxy_port))
    server.listen(5)
    
    print(f"[*] Listening on {proxy_host}:{proxy_port}")
    
    while True:
        client_socket, addr = server.accept()
        print(f"[*] Accepted connection from {addr[0]}:{addr[1]}")
        client_handler = threading.Thread(target=handle_client, args=(client_socket,))
        client_handler.start()

if __name__ == "__main__":
    main()
