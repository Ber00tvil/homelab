# Installing ElasticSearch

Elasticsearch is a platform for distributed search and analysis of data in real time. It is a popular choice due to its usability, powerful features, and scalability.

Here I'm gonna show U the steps I use for biulding muy own instance.

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update
sudo apt install elasticsearch -y
```
During the installation read the **output**. It contains a password for your user!

![image](https://github.com/Ber00tvil/homelab/assets/102535253/7fe0470d-d71c-4c11-b1d4-72a67d4d5d64)

# Configuring ElasticSearch

Now I'll edit `/etc/elasticsearch/elasticsearch.yml`.

It is a common practice to change the default port to something different just to make an attacker's life a little bit more annoying.

In my case the new port is **9999**.

Also I configured an interface Elastic will listen on.

`sudo vim /etc/elasticsearch/elasticsearch.yml`

![image](https://github.com/Ber00tvil/homelab/assets/102535253/9bc3ca9f-fb2c-4622-8746-d4c94af0d6a0)

Now let's start ElasticSearch!

`sudo systemctl start elasticsearch`

If you want ElasticSearch to start every time you boot the system type this:

`sudo systemctl enable elasticsearch`

# Securing Elastic Search

By default, Elasticsearch can be controlled by anyone who can access the HTTP API.

You can restrict the acces to a specific host, by using eg. Ubuntu’s default firewall, UFW.

_\* Be sure to add all ports, that you are using eg. 22_
```bash
sudo ufw allow from 192.168.0.164 to any port 9999
sudo ufw enable
sudo ufw status
```

![image](https://github.com/Ber00tvil/homelab/assets/102535253/0021c066-9a15-43ff-9565-cae2faa402f4)

# Connecting to ElasticSearch

Now it's time to verify if everything was set up correctly

`curl -X GET 'http://192.168.0.235:9999'`

And this will fail. By default Elastic requires HTTPS.

![image](https://github.com/Ber00tvil/homelab/assets/102535253/fadff29d-378b-4dac-93b3-8a2aa1291cf2)

We can see in logs, that conection is closed because of lack of HTTPS.

`sudo cat /var/log/elasticsearch/elasticsearch.log`

![image](https://github.com/Ber00tvil/homelab/assets/102535253/3126747a-88da-494e-b495-1869eb537cea)

If we add `-k` to ignore certificates and change protocol to **HTTPS**, now it will output, that we are 'missing authentication credentials for REST request.' 

![image](https://github.com/Ber00tvil/homelab/assets/102535253/b6ff3909-4943-433f-90f4-689d6c34bd62)

To solve this problem, we need to authenticate as user **elastic** with the password displayed when we were installing ElasticSearch.

![image](https://github.com/Ber00tvil/homelab/assets/102535253/dd63ad6c-8257-4cba-92ab-2ccb7f81f39e)

And now everything works!

# Installing Kibana

Kibana is a web interface for ElasticSearch.

_\* It will take a long time to install_

`sudo apt install kibana`

After installation we need to create enrollment ticket for kibana.

`sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana`

![image](https://github.com/Ber00tvil/homelab/assets/102535253/d83f1fdb-8dfd-4048-b5ba-3d13320664df)

This token configures authentication between Kibana and Elasticsearch.

Now to set up kibana run the script and paste in the token.

`sudo /usr/share/kibana/bin/kibana-setup`

![image](https://github.com/Ber00tvil/homelab/assets/102535253/fac2bca3-eb20-4bf0-8b91-36f715f6da45)

Now you can start Kibana (it takes a while) and enable it to start automatically whenever system boots. 

```bash
sudo service kibana start
sudo systemctl enable kibana
```

# Putting Kibana behin Nginx

It is always a good idea to put web services behind a web server like Nginx or Apache. So I'll install Nginx.

`sudo apt-get install nginx`

# Installing SSL certificate for Nginx

This command below will create a self-signed key and certificate pair with OpenSSL.

`sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-ssl-cert.key -out /etc/ssl/certs/nginx-ssl-cert.crt`

Don't forget to fill the '**Common Name**' prompt. You need to enter the domain name associated with your server or, more likely, your server’s public IP address.

![image](https://github.com/Ber00tvil/homelab/assets/102535253/7d8bed6e-bad1-4e93-8683-a046dc289fb4)

As an addition I'm gonna create Diffie-Hellman (DH) group, which is used in negotiating Perfect Forward Secrecy with clients.

`sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096`

# Creating a Configuration Snippet Pointing to the SSL Key and Certificate

`sudo nano /etc/nginx/snippets/self-signed.conf`

Within this file, you need to set the ssl_certificate directive to your certificate file and the ssl_certificate_key to the associated key. This will look like the following:

```bash
ssl_certificate /etc/ssl/certs/nginx-ssl-cert.crt;
ssl_certificate_key /etc/ssl/private/nginx-ssl-cert.key;
```

Next step is to define some SSL settings.

`sudo vim /etc/nginx/snippets/ssl-params.conf`

For our needs we will use already prepared settings from [https://cipherlist.eu/](https://cipherlist.eu/).

These settings may come at the cost of a compability. Thats why I'm going to remove one header. 

```bash
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/nginx/dhparam.pem; 
ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
ssl_ecdh_curve secp384r1;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# Disable strict transport security for now. You can uncomment the following
# line if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```
# Adjusting the Nginx Configuration to Use SSL

Create a domain in Nginx.

`sudo mkdir -p /var/www/test_lab_elastic.com/html`

Backup config file 

`sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak`
`sudo vim /etc/nginx/sites-available/default`

Map all domain requests that match **elastic.com** over port 443. Redirect all trafic from port 80 to 443.

```bash
server {
        listen 443 ssl;
        listen [::]:443 ssl;
        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        root /var/www/test_lab_elastc.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name test_lab_elastc.com www.test_lab_elastc.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

server {
    listen 80;
    listen [::]:80;

    server_name test_lab_elastc.com www.test_lab_elastc.com;

    return 302 https://$server_name$request_uri;
}
```

We need to adjust **ufw firewall**, so people can access the site.

![image](https://github.com/Ber00tvil/homelab/assets/102535253/77fc8e86-a8b5-413e-a85e-0e31955f5e72)

# Run Nginx

First, make sure there are no mistakes in configurations.

`sudo nginx -t`

This error is expected, don't worry.
![image](https://github.com/Ber00tvil/homelab/assets/102535253/e9b5d98f-b774-4e4f-ae44-68461a9308f8)

# Testing Encryption

Restart Nginx
`nginx -s reload`

Access the website

`https://server_domain_or_IP`

Upon visiting the site, warning will be displayed.

![Uploading image.png…]()
