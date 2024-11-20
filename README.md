threading.Thread(target=self.start_server)
        self.server_thread.start()

    def start_server(self):
        server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server_socket.bind(('0.0.0.0', self.port))
        server_socket.listen(5)
        print(f"Node listening on port {self.port}...")

        while True:
            client_socket, _ = server_socket.accept()
            data = client_socket.recv(1024)
            message = pickle.loads(data)
            self.process_message(message)

    def send_message(self, peer, message):
        client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        client_socket.connect(('127.0.0.1', peer.port))
        client_socket.send(pickle.dumps(message))
        client_socket.close()

    def process_message(self, message):
        if message['type'] == 'new_block':
            self.add_block(message['block'])

    def add_block(self, block):
        if block not in self.blockchain.chain:
            self.blockchain.add_block(block)
            print(f"New block added to blockchain by node {self.port}")
            # Propagate to other peers
            for peer in self.peers:
                self.send_message(peer, {'type': 'new_block', 'block': block})

# --------------------------- Simulation de l'utilisation ---------------------------
if __name__ == "__main__":
    # Initialisation des nœuds et du blockchain
    blockchain = Blockchain()
    node1 = Node(5000, blockchain)
    node2 = Node(5001, blockchain)

    # Création d'un portefeuille et ajout de transactions
    wallet = Wallet()
    address = wallet.get_address()
    print(f"Wallet Address: {address}")

    tx1 = Transaction(address, "Bob", 50)
    signed_tx1 = wallet.sign_transaction(tx1)
    blockchain.add_transaction(tx1)

    # Minage de blocs
    print("Mining block 1...")
    blockchain.add_block(Block(1, blockchain.get_latest_block().hash, time.time(), blockchain.pending_transactions))

    print("Mining block 2...")
    blockchain.add_block(Block(2, blockchain.get_latest_block().hash, time.time(), blockchain.pending_transactions))

    # Affichage de la blockchain
    for block in blockchain.chain:
        print("\n--- Block Info ---")
        print(f"Index: {block.index}")
        print(f"Previous Hash: {block.previous_hash}")
        print(f"Hash: {block.hash}")
        print(f"Data: {block.transactions}")
        print(f"Nonce: {block.nonce}")
