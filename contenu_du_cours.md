- 2 jours sur la gestion de configuration => Ansible (très simple à utiliser)

- 2 jours sur la partie plateforme virtuelle et cloud => openstack + un outil cloud (soit AWS ou GCP ou Azure, ou un outil générique)

- 2 jours sur les clusters (kubernetes ou mesos a priori plutôt kubernetes)

- 2 jours sur l'intégration continue, la scalabilité, etc... (gitlab)

Léger retour sur la durée de la thématique Gitlab. Certains pensent que la durée est trop longue, d’autres trop courte (propos tenus : Gitlab est une forge complète, incluant le ticketing, la gestion de sources, le CI/CD. 2 jours juste sur la partie CI/CD me paraissent suffisant, voir peu au vu des fonctionnalités.).

 
Trois niveaux :

- bare metal (matériel)

- virtualisation / cloud

- applicatif

Des domaines différents

- gestion de configuration / gestion des systèmes => ingénieur système

- gestion des plateformes virtuelles (cloud interne ou cloud externe) => ingénieur système et un peu développeur

- les outils de clustering => ingénieur système

- intégration continue / applicatif / containers => développeurs

Les outils classés par couche système

Par rapport aux outils cités, si on réordonne par couche système :

- Gestion de configuration

Ansible, puppet, chef, MS orchestrator

On pourrait ajouter des outils comme Vagrant qui ont eu un certain succès auprès des développeurs

- Construction de cloud

Open stack

(+ les outils VMWare et le software defined qui bougent beaucoup)

- Gestion de plateformes cloud

Les solutions propriétaires (AWS, GCP, Azure, etc...)

vrealize automation, morpheus

- Cluster et containers

Kubernetes

Mesos

Docker compose

- Intégration continue

Jenkins

Gitlab CI

 
Cursus sur 8 jours : 

- 2 jours sur la gestion de configuration => Ansible (très simple à utiliser)
- 2 jours sur la partie plateforme virtuelle et cloud => openstack + un outil cloud (soit AWS ou GCP ou Azure, ou un outil générique)
- 2 jours sur les clusters (kubernetes ou mesos a priori plutôt kubernetes)
- 2 jours sur l'intégration continue, la scalabilité, etc... (gitlab)

C'est très dense. C'est aussi possible de ne traiter que deux ou trois sujets sur les 4 (par exemple, ansible, docker, kubernetes, gitlab, ce qui fait déjà une bonne boîte à outils et est sans doute plus opérationnel)

Pré-requis
- Connaissances système
- Python
