# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :**
**Groupe :**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > On remarque 3 noeuds tous Ready : 
   -k3d-lab-server-0 "control-plane" : Il représente le cerveau il héberge l’API Server (point d’entrée ), le scheduler (qui décide sur quel nœud placer chaque pod), les controllers (boucle de réconciliation) et etcd (la base d’état du cluster).
   -k3d-lab-agent-0 <none> (worker)	Il exécute les pods applicatifs, via kubelet et le runtime containerd. 
   -k3d-lab-agent-1	<none> (worker)	Idem.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >Nous avons 4 conteneurs : 
   -k3d-lab-server-0, k3d-lab-agent-0 et k3d-lab-agent-1, tous basés sur l’image rancher/k3s:v1.35.5-k3s1 ;
   -k3d-lab-serverlb, basé sur ghcr.io/k3d-io/k3d-proxy.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >Le kubectl get all montre la chaîne suivante :
   -deployment.apps/web : c’est le seul objet que vous avez créé directement. Il décrit l’état désiré (1 réplica, image nginx).
   -replicaset.apps/web-68d995574f : créé automatiquement par le Deployment. Il maintient le nombre de pods voulu (DESIRED 1, CURRENT 1).
   -pod/web-68d995574f-hl9zp : créé automatiquement par le ReplicaSet. C’est lui qui exécute le conteneur nginx.
   
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > on voit le pod web-68d995574f-hl9zp passer en Completed (il est arrêté), et un nouveau pod web-68d995574f-szh2b apparaître en Pending, puis ContainerCreating, puis Running (1/1).

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non, le message n'a pas survécu : la page est redevenue « Welcome to nginx! ».
   > Le fichier modifié se trouvait dans le conteneur, dont le système de fichiers
   > est éphémère. Le nouveau pod repart de l'image nginx d'origine.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Le port-forward est attaché à un seul pod précis.
   > Ce pod ayant été supprimé, le tunnel n'avait plus de cible (erreur 502).
   > Il faut relancer le tunnel vers le nouveau pod.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >  `create` crée une ressource et échoue si elle existe déjà.
   `apply` la crée ou la met à jour selon le fichier YAML, on peut donc le relancer à chaque modification.

8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Il indique au Service quels pods viser grâce à leurs labels (ici `app: site`).
   S'il ne correspond à aucun pod, le Service n'a pas d'endpoints et le site est inaccessible.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   

   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    Le pod était en `ErrImagePull` puis `ImagePullBackOff`.
    `kubectl describe pod <nom-du-pod>` (section Events) a donné la cause :
    l'image `nginx:9.99` n'existe pas (`docker.io/library/nginx:9.99: not found`).
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    Oui, le site est resté en ligne.
    Lors d'un rolling update, Kubernetes ne supprime les anciens pods qu'une fois les nouveaux prêts.
    Le nouveau pod ne démarrant jamais, les 3 anciens pods ont continué à répondre.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://studious-space-bassoon-pgpqrq7jxqw27qpg-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >Oui, avec `kubectl get deploy mario` je retrouve le Deployment créé depuis Portainer.
    Portainer et `kubectl` parlent au même composant : l'API Server du cluster (sur le nœud control-plane).
    Ce ne sont que deux clients différents de la même API, donc ils voient les mêmes objets.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
