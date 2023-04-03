## 前提条件
- 首先我们准备好客户端申请文件csr和客户端私钥key，以及req.config，用csr和req.config去CA中心进行签署
- 通过CA证书中心去签署客户端csr，然后生成服务端证书和服务器端的私钥key
- 同时要有CA证书链（通常是包含多个证书的证书集合）

## nginx双向证书配置
```
listen 443 ssl;
        server_name www.xxx.com;
        ssl_certificate /data/program/nginx/conf/ecar-ssl/www.xxx.com.pem;
        ssl_certificate_key /data/program/nginx/conf/ecar-ssl/www.xxx.com.key;    
        ssl_client_certificate /data/program/nginx/conf/ecar-ssl/CA-Chain.pem;  #证书链
        ssl_verify_client on;
        ssl_verify_depth 3;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
```
## 客户端认证
```
openssl s_client -servername www.xxx.com -connect www.xxx.com:443 -state -CAfile CA-Chain.pem  -cert xxx.com.crt -key xxx.com.key
```
或者
```
openssl s_client -servername www.xxx.com -connect www.xxx.com:443 -state -CAfile CA-Chain.crt  -cert xxx.com.crt -key xxx.com.key
```
