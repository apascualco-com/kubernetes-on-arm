# kubernetes on arm v1.19

## Init cluster
En la carpeta ../cluster tendrás todo lo necesario para iniciar tu propio cluster de kubernetes

### Install continer and kubernetes files

Para la instalación del cluster, debes instalar el daemon que gestionará las imagenes de docker (runtime containerd) primero.

- sudo ./cluster/runtime-containerd.sh
- sudo ./cluster/kubernetes-install.sh

### Init cluster and join to master

Una vez instalado el cluster podemos iniciarlo a nuestro gusto, modificando el archivo de configuración

- sudo kubeadm init --config init-cluster.yaml
- editar join-cluster.yaml, con las credenciales, que ha devuelto init
- Repetir el proceso de instalación en el nodo y lanzar el join
- sudo kubeadm join --config join-cluster.yaml

## Metallb 
Instalar Metallb es altamente recomendado para trabajar conjutamente con ingress. Este "deploy", nos dara un loadbalancer que asignará según necesitamos una ip local

Podéis configurar el rango de ips y puesto si queréis, a vuestro gusto. Debería funcionar por defecto con esta configuración

Primero crearemos el namespace
- kubectl apply -f ./metallb/00-nasmespace.yaml
Se podría meter este secret en un fichero y lanzar todo el directorio, para la versión 2
- kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
Todo estará instalado bajo el namespace metallb-system
- kubectl apply -f ./metallb/*

## Ingress + nginx
Otra pieza bastante importante en nuestra arquitectura es ingress + nginx, estos nos ayudará mas adelante para aplicar los certificados ssl

kubectl apply -f ./ingress-nginx/*

## Certification manager
Esta carpeta nos instalará un deployment el cual gestionará los certificados que necesitamos de la mano de letsencrypt

Para certifiacados que utilizando el enpoint de producción de letsencrypt

- kubectl apply -f ./cert-manager/00-cert-manager.yaml
- kubectl apply -f ./cert-manager/01-production-config.yaml

Si querés hacer una prueba utilizado el enpoint de staging en vez de 01-production-config.yaml

- kubectl apply -f ./cert-manager/test-staging.yaml

Teneis que substituir TU-EMAIL-EN-LETSENCRYPT, por vuestro email en letsencrypt

## Ejemplo servicio (metallb + ingress + ssl)
En la carpeta ejemplo, tenéis el ejemplo de un servicio que nos permitrá redireccionar una peticion al host, especificado del puerto 80, hacia dentro de nuestro servidor al puerto 8080
