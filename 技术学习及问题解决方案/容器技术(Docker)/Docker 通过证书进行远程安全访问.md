## Docker通过证书进行远程安全访问（https://blog.csdn.net/alinyua/article/details/81086124）

```bash
# https://blog.csdn.net/alinyua/article/details/81086124
openssl genrsa -aes256 -out ca-key.pem 4096 
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=10.136.196.207" -sha256 -new -key server-key.pem -out server.csr
echo extendedKeyUsage = serverAuth >> extfile.cnf
echo subjectAltName = IP:10.136.196.207,IP:0.0.0.0,IP:127.0.0.1 >> extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
mv extfile.cnf extfile.cnf.old
echo extendedKeyUsage = clientAuth >> extfile.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem  -CAcreateserial -out cert.pem -extfile extfile.cnf
rm -v client.csr server.csr
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem
cp ca.pem server-* /opt/docker-ssl/
```

## 