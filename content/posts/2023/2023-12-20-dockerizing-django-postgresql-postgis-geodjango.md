---
title: Dockerizing Django with PostgreSQL, PostGIS and GeoDjango for Location Search
slug: Dockerizing Django with PostgreSQL, PostGIS and GeoDjango for Location Search
date: 2023-12-20
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.7/img/gallery/ca/ezra-jeffrey-comeau-adIeNK0Hbc4-unsplash.jpg
thumbnailImagePosition: left
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.7/img/gallery/ca/ezra-jeffrey-comeau-adIeNK0Hbc4-unsplash.jpg
categories:
- dev
tags:
- django
- postgresql
- postgis
- geodjango
- docker
draft: true
---


If you have stored the longitude and latitude coordinates for the address in your Django model and are using PostgreSQL with the PostGIS extension, you can leverage the spatial features of PostGIS to perform advanced geo searches similar to Elasticsearch. 

<!--more-->

{{< toc >}}


## Installation

Ensure you have installed the PostGIS extension for PostgreSQL and configured it for your database. You can refer to the PostGIS and Django documentation for installation instructions specific to your environment. Or follow the steps below for a shortcut.

### Install Docker

Follow the Docker documentation. In this example, we will install on Ubuntu. [^1]

[^1]: [Install using the apt repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

1. Set up Docker's `apt` repository.

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. Install the latest version of Docker packages.

```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verify the installation

```sh
sudo docker run hello-world
```

### Run PostgreSQL and pgAdmin in Docker

Run PostgreSQL and pgAdmin with Docker Compose. [^2]

[^2]: [PostgreSQL and pgAdmin](https://github.com/docker/awesome-compose/tree/master/postgresql-pgadmin)

```yaml
services:
  postgres:
    container_name: postgres
    image: postgres:latest
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PW}
      - POSTGRES_DB=${POSTGRES_DB}
    ports:
      - "5432:5432"
    restart: always
    volumes:
      - pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      - PGADMIN_DEFAULT_EMAIL=${PGADMIN_MAIL}
      - PGADMIN_DEFAULT_PASSWORD=${PGADMIN_PW}
    ports:
      - "5050:80"
    restart: always

volumes:
  pgdata:
    driver: local
```

Deploy with Docker Compose:

```sh
docker compose up
```

You might need to run the Docker as a non-root user [^3], otherwise it is possible to get `permission denied` error. 

[^3]: [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)

### Run PostgreSQL with PostGIS in Docker



### GeoDjango Installation

To install GeoDjango, there are several required steps as in [Django Documentation](https://docs.djangoproject.com/en/4.2/ref/contrib/gis/install/). The most important and the confusing step is to install Geospatial libraries. You can always trust and follow the steps in the [documentation](https://docs.djangoproject.com/en/4.2/ref/contrib/gis/install/geolibs/), however there is an easier way to install the required libraries and it's convenient for deployment of Django application.


## Define or update your model

Modify your Django model to include a `PointField` for storing the location coordinates. Here's an example:
    
```python
from django.contrib.gis.db import models

class Listing(models.Model):
    address = models.CharField(max_length=255)
    location = models.PointField()
```
    
Make sure to run the migrations to update your database schema after making this change.
    
## Perform geo searches

To search for listings within a certain distance from a given point, you can use the spatial functions provided by PostGIS. Here's an example:

```python
from django.contrib.gis.db.models.functions import Distance
from django.contrib.gis.geos import Point

def search_listings(request):
    # ... (previous code)
    user_location = Point(longitude, latitude, srid=4326)  # Create a Point object
    radius = 20  # Distance in kilometers
    listings = Listing.objects.annotate(distance=Distance('location', user_location)).filter(distance__lte=radius)
    # ... (continue with the result handling)
```

In the above code, the `annotate()` function adds a calculated field `distance` to each listing, representing the distance between the `location` field and the user's location. Then, the `filter()` function is used to retrieve the listings where the distance is less than or equal to the specified radius.

Note that the `srid=4326` parameter is necessary to specify the coordinate reference system (CRS) of the point. In this case, it corresponds to the WGS84 CRS commonly used for latitude and longitude coordinates.