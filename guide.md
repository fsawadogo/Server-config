
# ** Étape 1 : Télécharger et Installer OpenVPN**
### **1 Télécharger OpenVPN**
1. **Allez sur le site officiel d’OpenVPN** :  
   - [https://openvpn.net/community-downloads/](https://openvpn.net/community-downloads/)
2. **Téléchargez la version Windows Server** (Windows Installer).

### **1 Installer OpenVPN**
1. **Lancer l’installateur OpenVPN** en mode administrateur.
2. **Sélectionner les composants suivants** :
   - OpenVPN Service
   - EasyRSA (génération de certificats)
   - OpenVPN GUI
3. **Cliquer sur "Suivant" et installer**.
4. **Redémarrer le serveur après l’installation**.

---

# ** Étape 2 : Générer les Certificats et Clés**
### **1 Initialiser EasyRSA**
1. **Ouvrir une invite de commandes en mode administrateur**.
2. **Se rendre dans le dossier EasyRSA** :
   ```sh
   cd "C:\Program Files\OpenVPN\easy-rsa"
   ```
3. **Initialiser le PKI** :
   ```sh
   .\EasyRSA-Start.bat
   ./easyrsa init-pki
   ```

### ** Générer un certificat pour le serveur**
1. **Créer une autorité de certification (CA)** :
   ```sh
   ./easyrsa build-ca
   ```
   - Entrez un nom (ex: "OpenVPN-CA").

2. **Créer le certificat et la clé du serveur** :
   ```sh
   ./easyrsa build-server-full server nopass
   ```

3. **Créer le certificat Diffie-Hellman (DH)** :
   ```sh
   ./easyrsa gen-dh
   ```

4. **Générer un fichier de clé HMAC pour TLS Auth** :
   ```sh
   openvpn --genkey --secret ta.key
   ```

### **3️ Générer des certificats pour les clients**
1. **Créer un certificat pour chaque client** :
   ```sh
   ./easyrsa build-client-full client1 nopass
   ```
   - Répétez pour chaque utilisateur/client (ex: `client2`, `client3`...).

---

# **🔹 Étape 3 : Configurer OpenVPN sur le Serveur**
1. **Se rendre dans le dossier de configuration OpenVPN** :
   ```sh
   cd "C:\Program Files\OpenVPN\config"
   ```

2. **Créer un fichier `server.ovpn`** et ajouter la configuration suivante :
   ```ini
   port 1194
   proto udp
   dev tun
   ca "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\ca.crt"
   cert "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\issued\\server.crt"
   key "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\private\\server.key"
   dh "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\dh.pem"
   tls-auth "C:\\Program Files\\OpenVPN\\config\\ta.key" 0
   server 10.8.0.0 255.255.255.0
   ifconfig-pool-persist ipp.txt
   push "redirect-gateway def1 bypass-dhcp"
   push "dhcp-option DNS 8.8.8.8"
   keepalive 10 120
   comp-lzo
   persist-key
   persist-tun
   status openvpn-status.log
   verb 3
   ```

3. **Lancer le service OpenVPN** :
   - Ouvrez **Services.msc** et trouvez **OpenVPNService**.
   - Faites un clic droit et **démarrez le service**.

---

# ** Étape 4 : Ouvrir les Ports dans le Pare-Feu**
### **1️ Ajouter une règle pour OpenVPN**
1. **Ouvrir PowerShell en mode administrateur** et exécuter :
   ```sh
   New-NetFirewallRule -DisplayName "OpenVPN UDP 1194" -Direction Inbound -Protocol UDP -LocalPort 1194 -Action Allow
   ```

2. **Ajouter une règle NAT pour permettre l’accès à Internet** :
   ```sh
   netsh advfirewall firewall add rule name="NAT OpenVPN" dir=in action=allow protocol=ANY
   ```

---

# **🔹 Étape 5 : Configurer le Client OpenVPN**
### **1️ Transférer les fichiers nécessaires au client**
Copiez les fichiers suivants du serveur vers le client (PC distant) :
- `C:\Program Files\OpenVPN\easy-rsa\pki\ca.crt`
- `C:\Program Files\OpenVPN\easy-rsa\pki\issued\client1.crt`
- `C:\Program Files\OpenVPN\easy-rsa\pki\private\client1.key`
- `C:\Program Files\OpenVPN\config\ta.key`

### **2️ Installer OpenVPN sur le PC distant**
1. **Téléchargez OpenVPN** sur le PC distant (Windows, macOS, Linux).
2. **Ajoutez une configuration client** dans `C:\Program Files\OpenVPN\config\client1.ovpn` :
   ```ini
   client
   dev tun
   proto udp
   remote <IP-PUBLIQUE-SERVEUR> 1194
   resolv-retry infinite
   nobind
   persist-key
   persist-tun
   ca ca.crt
   cert client1.crt
   key client1.key
   tls-auth ta.key 1
   comp-lzo
   verb 3
   ```

### **3️ Se connecter au VPN**
- **Windows** : Ouvrir OpenVPN GUI, cliquer **Connect**.
- **Linux/macOS** : Utiliser `openvpn --config client1.ovpn`.

---

# **🔹 Étape 6 : Sécuriser le VPN**
### **1️ Activer l’authentification MFA**
- Ajoutez un module d’authentification à deux facteurs (ex: **Google Authenticator**).

### **2️ Bloquer l’accès si une connexion échoue plusieurs fois**
- Utilisez `fail2ban` pour bloquer les IP suspectes.

---

# **Conclusion**
Votre **serveur VPN OpenVPN sous Windows Server 2022 est maintenant opérationnel**!  
Vous pouvez vous **connecter en toute sécurité à votre réseau** depuis l’extérieur.