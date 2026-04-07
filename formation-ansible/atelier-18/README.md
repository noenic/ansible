# Atelier-18 Utilisation de templates Jinja 2

En se basant sur l'atelier 17 qui permettait de déployer un service NTP sur 4
machines : debian, suse, rocky et ubuntu

```console
.
├── chrony.yml
└── vars
    ├── chrony_debian.yml
    ├── chrony_opensuse-leap.yml
    ├── chrony_rocky.yml
    └── chrony_ubuntu.yml
```

Afin d'ajouter le chemin du fichier en haut du chrony.conf, nous mettrons en
place des templates.

Mis à disposition d'ansible dans le fichier templates, nous utiliserons pour
cela les variables existantes "`{{chrony_file}}`"


```console
.
├── chrony-01.yml
├── chrony-02.yml
├── templates
│   └── chrony.conf.j2
└── vars
    ├── chrony_debian.yml
    ├── chrony_opensuse-leap.yml
    ├── chrony_rocky.yml
    └── chrony_ubuntu.yml
```

Le fichier template `chrony.conf.j2` contient la configuration standard ntp avec en plus le
chemin du fichier de configuration une fois sur la machine :

```
# {{chrony_file}}
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```



Et éditerons le fichier chrony afin de lui faire viser utiliser le  fichier
template

```diff
     - name: Install custom configuration
-      copy:
-        dest: "{{ chrony_file }}"
-        content: |
-          # {{ chrony_file }}
-          server 0.fr.pool.ntp.org iburst
-          server 1.fr.pool.ntp.org iburst
-          server 2.fr.pool.ntp.org iburst
-          server 3.fr.pool.ntp.org iburst
-          driftfile /var/lib/chrony/drift
-          makestep 1.0 3
-          rtcsync
-          logdir /var/log/chrony
-      notify: Restart Chrony
+      template:
+        dest: "{{ chrony_file }}"
+        mode: 0644
+        src: chrony.conf.j2
```


Et une fois le playbook terminé, nous voici avec les fichiers de configuration
correctemnt configuré :

```console
vagrant@suse:~> uname -sn
Linux suse
vagrant@suse:~> sudo cat /etc/chrony.conf
# /etc/chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```
