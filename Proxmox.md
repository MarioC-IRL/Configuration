*Ouvréez un fichier ayant comme chemin /etc/nginx/sites-available/proxmox.conf.*
```bash
nano /etc/nginx/sites-available/proxmox.conf
```
*Ajouter le contenu suivant dans le fichier de conf.*
```yml
server {
    # Écoute sur le port 80 en IPv4/IPv6
    listen 80;
    listen [::]:80;

    # Définition de l'url utilisée pour accéder à l'hôte.
    server_name github.com;

    # Réécriture de l'URL utilisée pour l'hôte en HTTPS
    return 301 https://$host$request_uri;
}

server {
    # Écoute sur le port 443 en IPv4/IPv6
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    # Définition de l'url utilisée pour accéder à l'hôte.
    server_name github.com;
    
    # Configuration du journal d'accès et d'erreurs.
    access_log /var/log/nginx/proxmox.access.log;
    error_log  /var/log/nginx/proxmox.error.log;

    ssl on;
    ssl_certificate /etc/pve/local/pveproxy-ssl.pem;
    ssl_certificate_key /etc/pve/local/pveproxy-ssl.key;

    location / {
        # Définit le protocole et l'adresse vers notre hôte.
        proxy_pass https://localhost:8006;

        # La mise en mémoire tampon est désactivée.
        proxy_buffering off;
        # Nous définissons le paramétrage de la mémoire tampon.
        proxy_buffer_size 4k;

        # Définit la taille maximale autorisée des fichiers téléchargés par Nginx.
        client_max_body_size 5g;
        
        # Définit un délai d'attente pour établir une connexion avec votre serveur proxy.
        proxy_connect_timeout 300s;
        # Définit un délai d'attente pour la lecture d'une réponse du serveur proxy, l délai est défini uniquement entre deux  pérations de lecture successives.
        proxy_read_timeout 300s;
        # Définit un délai pour la transmission d'une requête au serveur proxy, le délai est fixé uniquement entre deux opérations d'écriture successives.
        proxy_send_timeout 300s;

        # Définit un délai pour la transmission d'une réponse au client. Le délai n'est fixé qu'entre deux opérations d'écriture successives.
        send_timeout 300s;

        # Définit la version du protocole HTTP pour le proxy.
        proxy_http_version 1.1;

        # Définit les informations d'en-tête au serveur proxy.
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Ssl on;

        # Permet de transmettre des champs d'en-tête.
        proxy_pass_header Authorization;
    }
}
```
*Ensuite, vous devez lier le fichier au répertoire des sites.*
```bash
ln -sf /etc/nginx/sites-available/proxmox.conf /etc/nginx/sites-enabled
```
```bash
systemctl edit nginx.service
```
*Ajouter les informations suivantes dedans.*
```yml
[Unit]
Requires=pve-cluster.service
After=pve-cluster.service
```
*Enregistrez le fichier et quittez le fichier, redémarrez le service nginx.*
