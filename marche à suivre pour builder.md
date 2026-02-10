Voici la marche à suivre pour builder et pousser l'image multi-arch sur Docker Hub :

1. Se connecter à Docker Hub
docker login -u fsiffert

2. Créer un builder multi-arch (une seule fois)
docker buildx create --name multiarch --use
docker buildx inspect --bootstrap

3. Build & push multi-arch (amd64 + arm64)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --build-arg POSTGRES_MAJOR_VERSION=17 \
  --build-arg POSTGIS_MAJOR_VERSION=3 \
  --build-arg POSTGIS_MINOR_RELEASE=5 \
  --build-arg PGVECTOR_VERSION=0.8.1 \
  -t fsiffert/postgis-vector:17-3.5-vector0.8.1 \
  -t fsiffert/postgis-vector:17-3.5 \
  -t fsiffert/postgis-vector:latest \
  --target postgis-prod \
  --push \
  .

--target postgis-prod est important car le Dockerfile a un stage postgis-test après — on ne veut pas inclure les dépendances de test dans l'image finale.

4. Vérifier sur Docker Hub
docker buildx imagetools inspect fsiffert/postgis-vector:latest

Cela affichera les manifests pour les deux architectures.

Pour votre déploiement Rancher
Dans votre workload, référencez simplement :

image: fsiffert/postgis-vector:17-3.5

K8s tirera automatiquement le bon manifest (amd64 ou arm64) selon l'architecture du node.