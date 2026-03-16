# ATELIER-01

## 1. Challenge 1

### 1.1 Démarrez la VM ubuntu

![image](./assets/capture1.png)
```bash
vagrant up ubuntu
```

### 1.2 Connectez-vous à la VM.

![image](./assets/capture2.png)
```bash
vagrant ssh ubuntu
```

### 1.3 Rafraîchissez les informations sur les paquets.
![image](./assets/capture3.png)
```bash
sudo apt update
```

### 1.4 Recherchez le paquet ansible.
![image](./assets/capture4.png)
```bash
apt search ansible
```

### 1.5 Installez le paquet officiel fourni par la distribution.
![image](./assets/capture5.png)
```bash
sudo apt install ansible
```
### 1.6  Notez la version d'Ansible.
![image](./assets/capture6.png)
```bash
ansible --version
```

### 1.7 Déconnectez-vous et supprimez la VM.
![image](./assets/capture7.png)
```bash
exit
vagrant destroy -f ubuntu
```


## 2. Challenge 2

### 2.1 Ajoutez le dépôt PPA pour Ansible.

![image](./assets/capture8.png)
```bash
sudo apt-add-repository ppa:ansible/ansible
```

### 2.2 Installez Ansible depuis ce dépôt.
![image](./assets/capture9.png)
```bash
sudo apt update && sudo apt install ansible
```

### 2.3 Notez la version d'Ansible.
![image](./assets/capture10.png)
```bash
ansible --version
```

> On voit bien que la version d'Ansible est plus récente que celle fournie par la distribution, `2.10.8` et `2.17.14` respectivement.

## 3. Challenge 3

Lancez une VM Rocky Linux et installez Ansible en utilisant PIP et Virtualenv.

Rocky Linux et Virtualenv

Notez bien que contrairement à Debian, le paquet python3-venv n'est pas nécessaire ici, étant donné que Virtualenv fait partie des modules standard de Python dans cette distribution.

### 3.1 Démarrez la VM Rocky Linux.
![image](./assets/capture11.png)
```bash
vagrant up rocky
```

### 3.2 Connectez-vous à la VM.
![image](./assets/capture12.png)
```
vagrant ssh rocky
```

### 3.3 Installez Python et les outils nécessaires.
![image](./assets/capture13.png)
```bash
sudo dnf install python3 python3-pip
```

### 3.4 Créez un environnement virtuel et activez-le.
![image](./assets/capture14.png)
```bash
python3 -m venv ansible-env
source ansible-env/bin/activate
```

### 3.5 Installez Ansible dans l'environnement virtuel.
![image](./assets/capture15.png)
```bash
pip install ansible && ansible --version
```
### 3.6 Voir la version d'Ansible installée.
![image](./assets/capture16.png)
```bash
ansible --version
```


