
# Private Node Deployment



### System Requirements

* OS: Linux
* CPU: Intel Core i3 or equivalent
* Storage: 200 GB internal storage drive
* Memory: 8 GB RAM
* Internet Speed: 10 Mbps
* A public IP address is strongly recommended



### Installation

**Step 1. Install docker. It is recommended to follow the instructions from the official site**

**Step 2. get the docker image**

```
docker pull jack0818/privateserver:latest
```

**Step 3. Run private node application**


```shell
sudo docker run -d  -p 8012:8012 -p 8013:8013  -e port=8012  -e listenWs=8013 -e ipfs=false   -e public=true -e FederationMode=true -e peer=/ip4/54.251.110.107/tcp/8082/ws/p2p/12D3KooWESGJGtydLPQ9ki6ikchMDGBrCyGHSKjhTAqiWGRhjbzG,/ip4/44.195.250.124/tcp/8082/ws/p2p/12D3KooWAC2FgzLwi6b2zyRkE6aCPY7f6H2Cn5RLYbkDsLuTcp2d  -e mainNet=true -e domain=privatenode.sending.network -e wssport=443  -e PrivateMode=true -e PrivateApiToken="your-admin-token" -v /fed/run/dendrite-p2p:/fed/run/dendrite-p2p jack0818/privateserver:latest
```


_**Description**_

`-listenWs` Specify the WebSocket listen port if you want to use the _WS_ protocol; otherwise, set it to -1. Make sure you set at least one of the `-listen` or `-listen`.

`-listen` Specify the port used for _TCP_ and _UDP_ if these protocols are in use; otherwise, set it to -1. Make sure you set at least one of the `-listenWs` or `-listen`.

`-port` Specify the port for receiving API requests. Set to -1 in most cases since the edge node is seldom used as a client.

`-ipfs` Set `true` to support the IPFS feature.

`-public` Set `true` if the server has a public IP address and port; otherwise, set `false`.

`-announce` Specify the public IP addresses and ports explicitly if applicable, separated by commas. If `-public` is set to `false` and no `-announce` value is provided, the node can only be reached by relays, leading to reduced performance and connectivity.

`-mainNet` Set `true` if the server join the main network; otherwise, set `false`.

`-domain` Use your server domain.

`-PrivateApiToken` your admin token, which will be used to Configure the DID-Whitelist of the PrivateNode. Detailed DID-Whitelist configuration can be found in step 5.

`-PrivateMode` Set `true` if the server private node; otherwise, set `false`.

`-PrivateMode` Set `true` if the server private node; otherwise, set `false`.

`-ProfileStoreMode` Specify the profile store mode. There are three modes: `remoteserver`,`localserver`,`offstoreserver`.  `remoteserver` is centralized storage. `localserver`  is local database.
   `offstoreserver` is decentralized storage.

**Step 4. Install Nginx**

1. Prepare a domain name.
2.  Configure a proxy to verify the certificate, convert the https protocol and forward it to the private server listen port.

    You can find more about _Nginx_ proxy configurations [here](https://phoenixnap.com/kb/how-to-install-nginx-on-ubuntu-20-04).

    You can apply for a certificate with _Cerbot_, detailed configuration can be found [here](https://certbot.eff.org/instructions?ws=nginx\&os=ubuntufocal).

Below is a sample _Nginx_ configuration:

```nginx
server {
    server_name fednode.sending.me;
    listen 80;
    listen [::]:80;

    access_log /var/log/nginx/privatenode.sending.network_access.log;
    error_log /var/log/nginx/privatenode.sending.network_error.log;

    location / {
       
        proxy_pass http://127.0.0.1:8012;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;

        client_max_body_size 50M;
        #add_header Access-Control-Allow-Methods "OPTIONS, GET, POST, HEAD, PUT, DELETE";
        #add_header Access-Control-Allow-Origin "*";
    }

}

server {
    server_name fednode.sending.me;
    
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    access_log /var/log/nginx/privatenode.sending.network_access.log;
    error_log /var/log/nginx/privatenode.sending.network_error.log;

    #listen [::]:443 ssl ipv6only=on; # managed by Certbot
    #listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/privatenode.sending.network/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/privatenode.sending.network/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
      
         
        proxy_pass http://127.0.0.1:8012;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;

        client_max_body_size 200M;
        #add_header Access-Control-Allow-Methods "OPTIONS, GET, POST, HEAD, PUT, DELETE";
        #add_header Access-Control-Allow-Origin "*";
    }
         
}
```
**Step 5. Configure the DID Whitelist of the PrivateNode**

  Only those DIDs in Whitelist can use the PrivateNode. Now we have not provided the Whitelist management page, only provided the following 3 network interfaces. Need to configure Whitelist each privateNode alone
  
1. Add a DID to the DID whiteList
   
    POST: https://{domain}/_api/client/did/whitelist?token="configured by admin"
   
    ```
    {"did":"355D738C10381e723b89aDa9Fda4aE59a988566g"}
    ```
    
3. Delete a DID from the DID whiteList
   
    DELETE: https://{domain}/_api/client/did/whitelist/{did}?token="configured by admin" 
   
4. Query the current PriveNode DID whiteList
   
    GET: https://{domain}/_api/client/did/whitelist?token="configured by admin"  
    RSP:
   
        ```
        {
        "dids": [
           "012621ef7d849cce12be0e59169779af86230290"
           ]
        }
        ```
      
