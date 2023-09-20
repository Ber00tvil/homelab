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

`sudo nano /etc/elasticsearch/elasticsearch.yml`

![image](https://github.com/Ber00tvil/homelab/assets/102535253/9bc3ca9f-fb2c-4622-8746-d4c94af0d6a0)

Now let's start ElasticSearch!

`sudo systemctl start elasticsearch`

If you want ElasticSearch to start every time you boot the system type this:

`sudo systemctl enable elasticsearch`

# Securing Elastic Search

By default, Elasticsearch can be controlled by anyone who can access the HTTP API.

You can restrict the acces to a specific host, by using eg. Ubuntu’s default firewall, UFW.

\* Be sure to add all ports, that you are using eg. 22

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
