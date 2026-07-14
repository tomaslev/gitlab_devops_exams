# GitLab CI/CD DevOps Exam — SkyBlue IT Limited

DevOps exam project: a complete GitLab CI/CD pipeline that tests, builds, and
deploys a three-service FastAPI application (gateway, users, orders) to a
k3s Kubernetes cluster with four environments.

## Architecture

- **3 microservices** (see original README below): an API `gateway` handling
  JWT auth and routing, plus internal `users` and `orders` services.
- **Images** are built from the Dockerfiles in each service directory and
  pushed to DockerHub: `<dockerhub-user>/exam-gateway`, `exam-users`,
  `exam-orders`, tagged with the commit SHA and `latest`.
- **Kubernetes deployment** via a single Helm chart (`charts/exam-app`)
  containing a Deployment + Service per microservice. The gateway is exposed
  as a NodePort; users and orders are internal (ClusterIP).
- **4 environments** as namespaces on one cluster: `dev`, `qa`, `staging`,
  `prod`, each on its own NodePort (30080–30083).

## Pipeline (`.gitlab-ci.yml`)

| Stage | Job | Behaviour |
|---|---|---|
| test | test-users | Runs the users service unit tests in a `python:3.7.7` container |
| build | build-images | Builds all three images, pushes to DockerHub (SHA + latest tags) |
| deploy_dev | deploy-dev | Automatic Helm deploy to `dev` |
| deploy_qa | deploy-qa | Automatic Helm deploy to `qa` |
| deploy_staging | deploy-staging | Automatic Helm deploy to `staging` |
| deploy_prod | deploy-prod | **Manual** deploy to `prod`, job exists **only on `main`** |

Each deploy runs `helm upgrade --install` with `--set image.tag=$CI_COMMIT_SHORT_SHA`,
so every environment runs exactly the images built by that pipeline.

Jobs run on a self-hosted shell runner (tag `exam-shell`) on the same EC2
instance that hosts the k3s cluster. DockerHub credentials are stored as
masked GitLab CI/CD variables (`DOCKER_USERNAME`, `DOCKER_PASSWORD`).

## Links

- DockerHub images: https://hub.docker.com/u/bonwitt
- GitLab project (pipelines): https://github.com/tomaslev/gitlab_devops_exams

---

*(Original application README below)*

# PROJET GITLAB DATASCIENTEST


# Microservices, API Gateway, Authentification avec FastAPI

- Ce référentiel est composé d'un ensemble de petits microservices prenant en compte l'approche de la passerelle API
- Le nombre prévu de microservices était de deux, mais étant donné que les services
   ne doivent pas créer de dépendance les uns sur les autres pour empêcher le SPOF, également pour éviter les codes en double,
   j'ai décidé de mettre une passerelle api devant qui fait l'authentification JWT pour les deux services
   dont je suis inspiré par Netflix/Zuul
- Nous avons 3 services dont une passerelle.
- Seule la passerelle peut accéder aux microservices internes via le réseau interne (utilisateurs, commandes)

## Services

- gateway : Construite au-dessus de FastAPI, simple passerelle api dont le seul devoir est de rendre propre
   routage tout en gérant l'authentification et l'autorisation
- utilisateurs (a.k.a admin): conserve les informations de l'utilisateur dans sa propre fausse base de données (système de fichiers).
   Peut être exécuté de simples opérations CRUD via le service. Il y a aussi un autre
   point de terminaison pour la connexion, mais le client est extrait de la réponse réelle. Ainsi, le service passerelle
   gérera la réponse de connexion et générera le jeton jwt en conséquence.
- commandes : Les utilisateurs (abonnés - authentification) peuvent créer et consulter (leurs - autorisations) commandes.

## Exécuttion
- check ./gateway/.env => 2 URL de services sont définies sur la base de la conf de douze facteurs
- docker-composer --build
- visitez l'adresse => http://localhost:8001/docs

# Exemples de requêtes
- Il existe déjà 2 utilisateurs dans la base de données des utilisateurs
- obtenir un jeton api avec l'utilisateur administrateur
  ```
  curl --header "Content-Type: application/json" \
       --request POST \
       --data '{"username":"admin","password":"a"}' \
       http://localhost:8001/api/login
  ```
- Vous verrez quelque chose de similaire à ci-dessous
  ```
  {"access_token":"***","token_type":"bearer"}
  ```
- utiliser ce jeton pour faire des demandes au niveau administratif
  ```
  curl --header "Content-Type: application/json" \
       --header "Authorization: Bearer ***" \
       --request GET \
       http://localhost:8001/api/users
  ```
- Des essais similaires peuvent également être effectués avec l'utilisateur par défaut pour créer et afficher les commandes


##Diagramme
![ScreenShot](https://github.com/DataScientest/gitlab_devops_exams/blob/main/diagram.png)

##Documentation Page
![ScreenShot](https://github.com/DataScientest/gitlab_devops_exams/blob/main/docs.png)
