# This step will be dedicated generating self-signed ssl certificate for run docker registry

## You have to generate certificate and private key

```
openssl req \
 -x509 -newkey rsa:4096 -days 365 -config openssl.conf \
 -keyout certs/domain.key -out certs/domain.crt
```

```
openssl x509 -text -noout -in certs/domain.crt
```

## Then do the following on all worker nodes.

## 1. Add insecure-registries

```
sudo touch /etc/docker/daemon.json
```

```
sudo nano /etc/docker/daemon.json
```

And write this

```
{ "insecure-registries" : ["local:registry:ip:5000"] }
```

Then

```
sudo systemctl restart containerd
```

```
sudo systemctl restart k3s
```

## 2. Add self-signed certificate to trust store

Copy your certificate on each node and do this:

```
sudo cp domain.crt /usr/local/share/ca-certificates
```

```
sudo update-ca-certificates
```

## 3. Add certificate to docker folder

```
sudo mkdir -p /etc/docker/certs.d/{registry_ip}:5000
```

```
sudo cp domain.crt /etc/docker/certs.d/{registry_ip}:5000/ca.crt
```


# When i try to install all depences i faced a lof off errors. Most headache gave me error ImagePullBackOff. And [this](https://stackoverflow.com/questions/74727327/insecure-docker-registry-and-self-signed-certificates) helped me.