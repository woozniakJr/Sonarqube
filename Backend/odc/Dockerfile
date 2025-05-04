# Utiliser une image Python officielle
FROM python:3.10-slim

# Définir le répertoire de travail dans le conteneur
WORKDIR /app

# Copier les fichiers dans le conteneur
COPY . /app

# Installer les dépendances
RUN pip install -r requirements.txt

# Exposer le port 8000
EXPOSE 8000

# Lancer l'application Django
CMD [ "gunicorn", "--bind", "0.0.0.0:8000", "odc.wsgi:application" ]