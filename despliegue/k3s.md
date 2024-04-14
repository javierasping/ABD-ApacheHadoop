## 2.1 Despliegue cluster de k3s

Para el despliegue de Apache Hadoop voy a hacerlo sobre un cluster de Kubernetes , en concreto sobre K3s . Contare con 1 nodo master y 4 workers .



### 2.1.1 Despliegue del nodo master

Instalaremos K3s con el siguiente comando :

```bash
root@cruces-k8s-1:~# curl -sfL https://get.k3s.io | sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.29.3+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.29.3+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.29.3+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

```

Ahora este nos habrá creado un token en el siguiente directorio :

```bash
root@cruces-k8s-1:~# sudo cat /var/lib/rancher/k3s/server/node-token 

K10ae0d9982a88ed07b4ed5d99e10a33dd5eba00cac0721ddfd0c41f1d4086aa560::server:df1969e7f674509c4523fd47fd5ffa0d
```

Todos los nodos workers que queramos unir al cluster de K3s , le tendremos que indicar este token .

### 2.1.2 Despliegue de los nodos workers 

Para añadir los workers a nuestro nodo de k3s , introduciremos el siguiente comando :

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

Asi que tenemos que tener en cuenta la IP del nodo master asi como el token , sustituiremos los valores por los propios de nuestro escenario :

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://172.22.200.131:6443 K3S_TOKEN=K10ae0d9982a88ed07b4ed5d99e10a33dd5eba00cac0721ddfd0c41f1d4086aa560::server:df1969e7f674509c4523fd47fd5ffa0d sh -
```

Ahora lanzaremos este comando sobre todos los nodos workers , una vez finalice su instalación podremos comprobar que se han unido al cluster con el siguiente comando :

```bash
root@cruces-k8s-1:~# kubectl get nodes
NAME           STATUS   ROLES                  AGE     VERSION
cruces-k8s-1   Ready    control-plane,master   8m20s   v1.29.3+k3s1
cruces-k8s-4   Ready    <none>                 51s     v1.29.3+k3s1
cruces-k8s-5   Ready    <none>                 48s     v1.29.3+k3s1
cruces-k8s-2   Ready    <none>                 39s     v1.29.3+k3s1
cruces-k8s-3   Ready    <none>                 33s     v1.29.3+k3s1
```

Con esto realizado ya tendremos una configuración básica de un cluster de k3s .

