 openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server-key.pem -out server.pem -subj "/CN=ingress.almuhametov.com/O=almuhametov.com"
 kubectl create secret tls tls-secret --key server-key.pem --cert server.pem
 
curl -H "Host: dens-al.pro" 
