# __DevOps Day 1 - Discovery__

## __Questions step 1__

### Quelles sont les informations que l'on retrouve dans ce fichier ?

Le fichier de configuration contient les informations nécessaires pour que l'utilisateur se connecte à un cluster Kubernetes, y compris les détails des clusters, des contextes, des utilisateurs et des préférences.

### Quelle est la différence ?

Je ne peux pas accéder à la liste de pods car je n'ai pas les autorisations pour accéder au namespace default en tant que theo-gimond.

### Quelles sont les propriétés principales que l'on retrouve ?

On retrouve les propriété du pod telles que :
La version de l'API (V1)
Les informations principales du pod (label, nom, namespace...)
Les spécifications du pod (conteneurs, ressources, volumes...)
Et le status (conditions de démarrage, IP du pod, et status du conteneur)

Ici, le pod "mynginx" utilise l'image registry.takima.io/school/proxy/nginx et est en cours d'exécution. 

### Que se passe-t-il lors de cette deuxième création en impératif ?

La deuxième exécution de la commande est impossible parce qu'un pod a déjà été créé avec le même nom que celui qu'on essaye de créer. 

### Que se passe-t-il lors de cette deuxième création en déclaratif ?

Un message apparaît qui propose d'appliquer la configuration du .yaml au pod existant. 
La création en déclaratif ne crée pas de deuxième pod du même nom. 

### Que remarquez-vous dans la description des properties spec: template ? À quoi sert le selector: matchLabels ?

Dans les spec, on retrouve le nom du pod et ses labels et le nom du conteneur et de l'image utilisés pour créer le pod.
le selector matchLabels permet de faire correspondre les labels des pods avec ceux spécifiés dans le selector. Ici, il sélectionne les pods ayant comme label l'app unicorn-front et les réplique 3 fois.

### Combien y a-t-il de pods déployés dans votre namespace ?

Maintenant, il y a le pod d'origine mynginx ainsi que les 3 replicasets. Il y a donc 4 pods. 

### Suppression du pod Que se passe-t-il ?

Le pod est bien supprimé mais les 3 autres sont toujours actifs.

### Suppression du ReplicaSet que se passe-t-il ? 

Tous les pods issus du replicaset sont supprimés. 

### Quels sont les changements par rapport au ReplicaSet ?

Le deployment n'utilise pas la même image que le ReplicaSet. La principale différence est que le deployment propose une gestion des versions eds replications tandis que le replicaset se contente de répliquer les pods existants. 
Aussi, le deployment nécessite la spécification d'un port (80).

### Deployment appied : Combien y a-t'il de ReplicaSet ? De Pods ?

Une fois le déploiement effectué, on voit qu'un ReplicaSet est actif et que 3 pods ont été créés à partir de ce ReplicaSet.

### Une fois terminé, combien y a-t-il de replicaset ? Combien y a-t-il de Pods ? Allez voir les logs des événements du déploiement avec kubectl describe deployments. Qu’observez-vous ?

Après le rollback, on voit 2 replicaset correspondant aux 2 versions de l'image, 3 pods qui ont été recréés à partir de l'ancien replicaset. 
Dans les logs du deployment, on peut voir les versions par lesquelles sont passés les pods. (to 1 from 2)

### Mise à jour du Deployment : Que se passe-t-il ? Pourquoi ?

On voit que 3 pods ont été mis à jour et sont actifs grâce au replicaset spécifié dans le deployment. 
Mais un pod n'arrive pas à se mettre à jour ce qui crée un problème. 

### Combien y a-t-il de révisions ? À quoi correspond le champ `CHANGE-CAUSE` ?

On peut voir qu'il y a 5 versions. Nous sommes à la 5ème qui pose problème et nous souhaitons donc revenir à une version précédente.
Avec la commande : 
```sh
kubectl rollout history deployment.v1.apps/unicorn-front-deployment --revision=4
```
on peut voir le contenu de la version 4 et vérifier que nous souhaitons bien revenir à cette dernière. 
Nous revenons à la version 4 grâce à la commande : 
```sh
kubectl rollout undo deployment.v1.apps/unicorn-front-deployment --to-revision=4
```

### Combien y a-t'il de Pods?

Il y a maintenant 5 pods. 

### Deployment paused : Que se passe-t-il au niveau ReplicaSet ?

Les ReplicaSet ne s'activent pas pour changer la version des pods. Le ReplicaSet actif est donc celui qui avait été activé à la version d'avant. 

### Deployment resumed : Que se passe-t-il au niveau ReplicaSet ?

La version indiquée après le pause (20.2) est activée et appliquée à 5 nouveaux pods. 

## Bonus 

### Que constatez-vous ? Pourquoi ?

Le pod a été kill parce qu'il a une limite de mémoire de 100Mo mais l'application en demande 250 ce qui est supérieur à la limite. 

### CPU-LIMIT Que constatez-vous ? Pourquoi ?

Le pod utilise presque toute la capacité CPU qui lui est allouée. Son utilisation mémoire est très faible. La capacité CPU se calle sur la limite au lieu de prendre le cpu indiqué par l'application.


## __Questions step 2__

### Première version de hello-deployment : Que se passe-t-il ? Pourquoi ?

Les images ne peuvent pas être récupérées car elles appartiennent à une registry privée et que nous n'avons pas indiqué de mot de passe pour accéder à la ressource. 

### Décrivez ce que répond la Web App ? Actualisez votre page avec CTRL + F5. Que se passe-t-il ?

La Web App renvoie le contenu de l'application. À chaque fois qu'on recharge la page, elle s'affiche dans une nouvelle couleur.

### Configs de variables d'environnemnt: Que constatez-vous sur le navigateur ?

Des informations supplémentaires ont été ajoutées grâce à l'ajout des variables d'environnement. Nous avons maintenant accès au nom de la node et du pod ainsi que l'IP du pod. 

### Bonus 3 : Que constatez vous ? 

On voit que quand la Target dépasse les 50%, des ressources sont allouées à l'application pour qu'elle puisse s'adapter.
Une fois les ressources effectives, on peut voir qu'elles repassent sous les 50%.

### Bonus 4 : Que constatez vous ? 

La connexion échoue car la politique de réseau limite les accès et la busybox n'a pas le label access=True. 

### Bonus 5 : Qu'est-ce qu'une Avaibility Zone au niveau infrastructure ?

Une Availibility Zone est une localisation distincte dans une interface cloud. Elles sont isolées les unes des autres pour garantir une haute disponibilité et elles sont connectées entre elles par des réseaux à faible latence pour permettre une redondance et une réplication des données. 


# __DevOps Day 2 - Deep Dive__

## __Questions step 1__

### Que se passe-t-il au niveau des Pods de l’API ? Vous pouvez jeter un œil aux logs. (kubectl logs -f nomdupod)

L'API n'arrive pas à se connecter à la base de données car nous n'avons pas configuré les bons identifiants pour s'y connecter. 

## __Questions Step 3__

### Quel est le nom du service de la base de données ?

Le nom du service est pg-service comme indiqué dans le fichier du même nom.

## __Questions Step 5__

### Pourquoi plus rien ne fonctionne ? Pourquoi faut-il kubectl rollout restart deployment MON_API

On a supprimé le pod de la base de données. Un nouveau pod est généré mais son adresse IP est différente. 
L'API se base sur l'adresse IP de l'ancien pod de la base de données et ne reconnait donc pas l'adresse IP du nouveau pod. 
Il faut donc rollout restart le déploiement de l'API pour qu'il se base sur l'adresse IP du nouveau pod de la base de données. 
On voit aussi que les données ajoutées dans la base de données sont supprimées. On cherche donc à rendre les données persistantes. 

## __Questions Step 6__

### D’après le tableau, quel est le type d’accès implémenté par notre Storage Class EBS ? Pourquoi cela convient parfaitement pour la persistance de la base de données Postgres ?

Le type d'accès implémenté par la storage class EBS est ReadWriteOnce d'après le tableau. 
Cela convient car un seul noeuf peut monter un volume en lecture/ecriture à la fois ce qui garantit l'intégrité des données et éviter les conflits d'accès. 

### Vérifiez que le PVC est créé avec le PV. Quel est le nom du PV ?

Le PVC est bien créé et porte le nom : pvc-ddf811a2-6298-499c-b106-1d3ead80059f

# __DevOps Day 3 - GitOps__

## Cluster ELK à 3 noeuds : Que constatez-vous ?

On remarque les 3 noeuds ont des utilisations différentes du disque et chaque noeud propose de l'espace de disque.