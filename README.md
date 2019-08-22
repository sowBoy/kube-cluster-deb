# Debian VMs ready kubernetes cluster



Un ensemble de playbooks pour mettre en place un cluster kubernetes avec des VMs debian9.9 

### Actions:

Après avoir cloné ce repositorie sur la machine ansible .

1. Installation de ansible sur la machine ansible

Simple avec pip :

`sudo apt update`
`sudo apt install python3-pip`
`pip3 --version`

Ensuite on install Ansible :

`pip3 install ansible`

2. Clefs ssh 

Générer une clef ssh sur la machine ansible avec `empty passphrage` et deployer la clef public dans
les machines du cluster kubernetes pour le user root de ces machines . 

 `cat $HOME/.ssh/id_rsa.pub` pour afficher la clef public dans le terminal, la copier puis la coller dans le
fichier `/root/.ssh/authorized_keys` sur chacune des machines du cluster kubernetes . 

3. Execution des playbooks 

NB: Avant de lancer un quelconque playbook, il faut être sûr de s'être connecté au moins une fois en ssh depuis la machine ansible aux machines du clusters en tant
que le user avec lequel vous executez les playbooks .
Ceci permets de mettre à jours le fichier `$HOME/.ssh/known_hosts` . Il faut également être sûr que l'utilisateur
exécuteur des playbooks soit propriétaire du fichier `$HOME/.ssh/known_hosts` .

Vous pouvez maintenant enregistrer les machines dans le fichier /etc/hosts de la machine ansible :

`
XX.XX.RR.XX  master
YY.XX.FF.FF  workerone
EE.EE.KK.XX  workertwo
`

NB: rentrer les adresses IP des machines à la place des chaines de caractères mais garder le nom des machine .

Il ne reste plus que à lancer les playbooks:

En étant à la racine du dossier Ansible_kubernetes lancer `ansible-playbook -i hosts nomDuFichier` .

Remplacer `nomDuFichier` par les playbooks dans cet ordre :

1. initial.yml  qui crée le user devops sur chaque machine du cluster avec deploiement de la clef ssh
2. kube-dependencies.yml qui disable swapp et install docker et les composants kubernetes(kubelet, kubeadm) sur chaque machine et kubectl sur master uniquement
3. master.yml initialisation du cluster et de son network
4. workers.yml rajout des workers au cluster

Note : si cette erreur "Failed to set permissions on the temporary files Ansible needs to create when becoming an unprivileged user" est rencontré il faut créer un fichier ansible.cfg sous
/etc/ansible/ et copié les deux lignes suivants:

[defaults]

allow_world_readable_tmpfiles=true


#### Tester que tout marche
  
Une fois tout les playbooks lancés pour vérifier que tout marche, se connecter en ssh au master avec le user devops :
`ssh devops@master` puis `kubectl get nodes` le rendu devra être comme ceci :
 
`
master      Ready    master   3d20h   v1.14.3
workerone   Ready    <none>   3d20h   v1.14.3
workertwo   Ready    <none>   3d20h   v1.14.3
`
Le master et les workers ont le statut ready tout fonctionne.

#### Faire tourner une application dans le cluster

Dépuis le master :

Déployons nginx dans notre cluster `kubectl create deployment nginx --image=nginx` .
Puis exposons l'application nginx deployée `kubectl expose deploy nginx --port 80 --target-port 80 --type NodePort` .

En lançant `kubectl get services` on devrait avoir un rendu comme ci-dessous :

`NAME         TYPE        CLUSTER-IP       EXTERNAL-IP           PORT(S)             AGE
kubernetes   ClusterIP   XX.YY.YY.Y        <none>                443/TCP             1d
nginx        NodePort    GG.GG.TT.OOP   <none>                80:nginx_port/TCP   40m`

Il de ne reste plus qu'a visiter l'url `http://workerone:nginx_port` ou `http://workertwo:nginx_port` pour voir le welcome page de nginx .

Vous pouvez supprimer l'application avec `kubectl delete service nginx` puis `kubectl delete deployment nginx` .

