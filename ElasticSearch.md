# Setting up ElasticSearch

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

Now I'll edit /etc/elasticsearch/elasticsearch.yml.

It is a common proactice to change the default port to something different just to make an attacker's life a little bit more annoying.

In my case the new port is **9999**.

Also I configured an interface Elastic will listen on.

`sudo nano /etc/elasticsearch/elasticsearch.yml`

![image](https://github.com/Ber00tvil/homelab/assets/102535253/9bc3ca9f-fb2c-4622-8746-d4c94af0d6a0)

