This dir contains plain yaml for creating router network without the
use of the operator. Modify the yaml by replacing the placeholders
CLOUD1.FQDN and CLOUD2.FQDN with public addresses of the routes
exposed on cloud1 and cloud2. (Note this assumes cloud1 and cloud2
(and edge1) are using openshift and the edge2 and edge3 are using some
other kubernetes distribution. May need mofifications for different
topologies).

To create TLS certs do something like:

mkdir -p internal-certs

# First we create the private key and self-signed certificate for the CA:
openssl genrsa -out internal-certs/ca-key.pem 2048
openssl req -new -batch -key internal-certs/ca-key.pem -out internal-certs/ca-csr.pem
openssl x509 -req -in internal-certs/ca-csr.pem -signkey internal-certs/ca-key.pem -out internal-certs/ca.crt

# Then we create a private key and certificate, signed by the CA for inter-router connections:
openssl genrsa -out internal-certs/tls.key 2048
openssl req -new -batch -subj "/CN=inter-router-demo" -key internal-certs/tls.key -out internal-certs/server-csr.pem
openssl x509 -req -in internal-certs/server-csr.pem -CA internal-certs/ca.crt -CAkey internal-certs/ca-key.pem -out internal-certs/tls.crt -CAcreateserial

then in each cluster:

kubectl create secret generic inter-router-certs --from-file=tls.crt=internal-certs/tls.crt  --from-file=tls.key=internal-certs/tls.key  --from-file=ca.crt=internal-certs/ca.crt
