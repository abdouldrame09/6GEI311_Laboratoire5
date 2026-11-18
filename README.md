# Laboratoire 5 : Déploiement avec distribution de charge (Load Balancing)

**Cours :** 6GEI311 — Architecture des logiciels  
**Session :** Automne 2026  
**Établissement :** Université du Québec à Chicoutimi (UQAC)  

---

## Membres de l'équipe
- **Dramé, Abdramane** (Code permanent : `DRAA04070300`) — *Membre A* (Branche : `AD`)
- **Doumbia, Djan Ibrahim** (Code permanent : `DOUD23110200`) — *Membre B* (Branche : `ib`)

---

## 1. Résumé des apprentissages

Dans ce laboratoire, nous avons mis en pratique les concepts fondamentaux du déploiement d'applications distribuées et de la haute disponibilité :

1. **Rôle d'un Répartiteur de charge (*Load Balancer*) et Reverse Proxy :**
   - Nous avons utilisé **Nginx** placé en amont de plusieurs instances de notre application web Flask.
   - Nginx intercepte l'ensemble des requêtes clientes sur un port unique (`8181`) et les distribue de manière transparente aux différents serveurs d'arrière-plan (*upstream*).

2. **Algorithme d'équilibrage Round-Robin :**
   - Nous avons observé comment Nginx alterne séquentiellement et équitablement le trafic entre l'instance du Membre A (`172.20.10.2:3000`) et celle du Membre B (`172.20.10.3:3001`), empêchant la surcharge d'un serveur individuel.

3. **Tolérance aux pannes (*Failover* / Haute disponibilité) :**
   - Lorsqu'une des instances applicatives subit un arrêt (panne matérielle, arrêt de processus ou crash), Nginx détecte automatiquement l'indisponibilité du nœud défaillant et route l'intégralité des nouvelles requêtes vers les nœuds encore opérationnels sans interruption visible pour le client.

---

## 2. Configuration de l'infrastructure

### Configuration Nginx (`conf/nginx.conf`)
Le bloc `http` a été configuré avec le cluster `labcluster` regroupant les deux serveurs :

```nginx
upstream labcluster {
    server 172.20.10.2:3000;
    server 172.20.10.3:3001;
}

server {
    listen 8181 default_server;

    location / {
        proxy_pass http://labcluster;
        try_files $uri $uri/ =404;
    }
}
```

### Détail des instances déployées :
* **Instance Membre A :**
  * Machine : `172.20.10.2`
  * Port : `3000`
  * Réponse : `<h2>Hello, World! From AD: 172.20.10.2</h2>`
* **Instance Membre B :**
  * Machine : `172.20.10.3`
  * Port : `3001`
  * Réponse : `<h2>Hello, World! From AD: 172.20.10.3</h2>`

---

## 3. Réponses aux questions de la manipulation

### Question — Étape 6 : Arrêt direct de l'application Flask
> **Instruction :** *Arrêtez l’application python, testez avec la même adresse sur votre navigateur. Qu’est-ce que vous remarquez ?*

* **Réponse :**  
  En accédant directement à l'adresse du serveur sans passer par un proxy (`http://172.20.10.2:3000/`) alors que l'application Python est arrêtée, le navigateur affiche une erreur de connexion (ex. `ERR_CONNECTION_REFUSED`). Aucun service n'étant à l'écoute sur le port `3000`, la requête échoue immédiatement.

---

### Question — Étape 16 : Accès via Nginx et rafraîchissements multiples
> **Instruction :** *Ouvrez le navigateur sur une machine quelconque, et naviguez vers l’adresse : `http://IP_MEMBRE_A:8181/`, faites des refresh plusieurs fois, qu’est-ce que vous remarquez ?*

* **Réponse :**  
  En rafraîchissant la page à plusieurs reprises, le message affiché alterne successivement :
  * Un rafraîchissement affiche : `Hello, World! From AD: 172.20.10.2` (réponse du Membre A)
  * Le rafraîchissement suivant affiche : `Hello, World! From AD: 172.20.10.3` (réponse du Membre B)
  
  Cela prouve que Nginx distribue la charge de façon équitable entre les deux instances en suivant la méthode **Round-Robin**.

---

### Question — Étape 17 : Arrêt de l'application du Membre A et tolérance aux pannes
> **Instruction :** *Arrêtez l’application du membre A sur la machine du membre A, est-ce que l’application est toujours accessible via :*
> * *a. `http://IP_MACHINE_A:3000/` ?*
> * *b. `http://IP_MEMBRE_A:8181/` ?*

* **Réponse :**  
  * **a. Via l'accès direct (`http://172.20.10.2:3000/`) : NON.**  
    L'application n'est plus accessible en direct car le processus local sur le port 3000 est éteint.
  * **b. Via le load balancer (`http://172.20.10.2:8181/`) : OUI.**  
    L'application reste parfaitement accessible. Nginx détecte que l'instance du Membre A ne répond plus et réachemine automatiquement 100 % des requêtes vers l'instance active du Membre B (`172.20.10.3:3001`). Le service ne subit donc aucune coupure pour les utilisateurs.
