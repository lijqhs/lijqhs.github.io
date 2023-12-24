---
title: Dockerizing Django with PostgreSQL, PostGIS and GeoDjango for Location Search
slug: Dockerizing Django with PostgreSQL, PostGIS and GeoDjango for Location Search
date: 2023-10-20
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
---


If you have stored the longitude and latitude coordinates for the address in your Django model, you can leverage the spatial features of PostGIS and GeoDjango to perform advanced geo searches similar to Elasticsearch. 

<!--more-->

{{< toc >}}


## Introduction



### Install Django

Install within a virtual environment and activate the environment, to deactivate just run `deactivate`.

```shell
python -m venv djvenv       # create venv named djvenv
source djvenv/bin/activate  # activate djvenv
pip install django==4.2
```

After Django is installed, create a Django project and an application.

```shell
django-admin startproject djsite
cd djsite
python manage.py startapp djapp
```

To test project is ok, run `python manage.py runserver`, and you will see something like

```
Django version 5.0, using settings 'djsite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

And Django will create a `db.sqlite3` in the directory of project for the purpose of development. The default setting for database is sqlite3, which is defined in `setting.py`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

In our example, we will replace the database to PostgreSQL.

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

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verify the installation

```shell
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

Create `.env` file in the same directory:

```shell
POSTGRES_USER=admin
POSTGRES_PW=changeit
POSTGRES_DB=pg4django
PGADMIN_MAIL=admin@example.com
PGADMIN_PW=changeit
```

Deploy with Docker Compose:

```shell
docker compose up
```

You might need to run the Docker as a non-root user [^3], otherwise it is possible to get `permission denied` error. 

[^3]: [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)

While the docker is running, get access to pgAdmin on `http://localhost:5050`. To register the PostgreSQL run by Docker as above in pgAdmin, right click `Servers` and choose `Register` -> `Server` -> `Connection`, set `Host name/address` to `postgres` (Docker container name).


### Replace database in Django to PostgreSQL

To connect PostgreSQL DB, it's better to make sure [psycopg](https://www.psycopg.org/install/) (3 or 2) has been installed, for this example, install psycopg 2:

```shell
pip install psycopg2-binary
```

Replace definition of `DATABASES` in `settings.py` with the following:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get("PG_DB_NAME"),
        'USER': os.environ.get("PG_USER"),
        'PASSWORD': os.environ.get("PG_PSWD"),
        'HOST': os.environ.get("PG_HOST_DEV" if DEBUG else "PG_HOST"),
        'PORT': os.environ.get("PG_PORT"),
    }
}
```

And create `.env` in the same directory of `manage.py` for environment definitions:

```shell
PG_USER=admin
PG_PSWD=changeit
PG_DB_NAME=pg4django
PG_HOST_DEV=localhost   # cannot be pg host when run django within docker
PG_HOST=xxx.xxx.xxx.xxx # local ip for postgres
PG_PORT=5432
```

You might need to install `dotenv` library to load those environment:

```shell
pip install python-dotenv
```

Now if run `python manage.py runserver`, the Django project should be run on PostgreSQL DB.

### Create models and admin page

Add `djapp` to `djsite` project in `settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'djapp',
]
```

Create model `Restaurant` in `models.py`:

```python
class Restaurant(models.Model):
    CHOICES_CUISINE_TYPE = [
        ('Asian food', 'Asian food'),
        ('European cuisine', 'European cuisine'),
        ('Cuisine of the Americas', 'Cuisine of the Americas'),
        ('African cuisine', 'African cuisine'),
    ]
    brand = models.CharField(verbose_name='Brand Name *', max_length=256)
    cuisine_type = models.CharField(max_length=128, choices=CHOICES_CUISINE_TYPE, null=True, blank=True)
    address = models.CharField(verbose_name='Address *', max_length=256)
```

And register the model in `admin.py` to make it available for management in Django administration page.

```python
from django.contrib import admin
from .models import Restaurant

@admin.register(Restaurant)
class RestaurantAdmin(admin.ModelAdmin):
    list_display = (
        'brand',
        'cuisine_type',
        'address',
    )

    fieldsets = [
        ('Basic Information', {
            'fields': (
                'brand',
            )
        }),
        ('Cuisine Information', {
            'fields': (
                'cuisine_type',
            )
        }),
        ('Location Information', {
            'fields': (
                'address',
            )
        })
    ]
```

Before access to the admin page, one should migrate and model into database and create an admin account:

```shell
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

Then run `python manage.py runserver`, enter `http://127.0.0.1:8000/admin` in browser to view the Django administration page. If everything works, you can see `Restaurants` under `DJAPP` column in the left side of the page.

### Deploy Django project with Docker

Docker can make deployment easier, especially to install Geospatial libraries for GeoDjango. Before run Django project in Docker, it is better to generate a `requirements.txt` from current virtual environment for Django:

```shell
pip freeze > requirements.txt
```

To better deploy the Django project, always follow the right steps and requirements. To check what might be missing for the deployment of your Django project, run `python manage.py check --deploy`. [^4] 

[^4]: [Django Tutorial Part 11: Deploying Django to production](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Deployment)

This section is to explain how to run Django project within Docker, which is a necessary step to explain how to install GeoDjango with Docker for this Django project. So right now, let us skip steps to deploy with WSGI[^5], CSRF security settings[^6], or even cache middlewares[^7], etc, which we should care about whenever deploy a real Django project.

[^5]: [How to deploy with WSGI](https://docs.djangoproject.com/en/4.2/howto/deployment/wsgi/)
[^6]: [How to use Django’s CSRF protection](https://docs.djangoproject.com/en/4.2/howto/csrf/)
[^7]: [Django’s cache framework](https://docs.djangoproject.com/en/4.2/topics/cache/)

Define `dockerfile` as follows:

```dockerfile
FROM python:3.10-slim-buster

WORKDIR /app

ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

COPY . .
RUN python -m pip install --upgrade pip && \
    pip3 install -r requirements.txt 
```

And `compose.yml`:

```yml
version: "3"
services:

  djapp:
    build:
      context: .
      dockerfile: dockerfile
    image: djsite
    command: python manage.py runserver 0.0.0.0:1024
    ports:
      - "1024:1024"
    volumes:
      - ./:/app/
    env_file:
      - ./.env
    restart: always
```

Use docker compose to run the Django project:

```shell
docker compose up
```

## Preparation for Location Search in Django

To do location search in Django, we need GeoDjango and PostGIS. 
- [GeoDjango](https://docs.djangoproject.com/en/4.2/ref/contrib/gis/tutorial/) is a web framework for developing geographic applications using Django, which is a high-level Python web framework.
- [PostGIS](https://postgis.net/), on the other hand, is an extension for the PostgreSQL database that adds support for spatial data types and spatial querying capabilities.


### Run PostgreSQL with PostGIS in Docker

It is easy to install PostGIS in PostgreSQL for this example, actually we don't need to install manually. All we have to do is to change the Docker image of PostgreSQL to be a PostGIS-ready image: [kartoza/postgis](https://hub.docker.com/r/kartoza/postgis).

```yml
services:
  postgres:
    container_name: postgres
    image: kartoza/postgis:latest
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

Before we build and run PostgreSQL with PostGIS in Docker and replace `PostgreSQL` in Django with `PostgreSQL with PostGIS`, it's better to clear previous docker container and volume and then rebuild docker image:

```shell
docker rm postgres
docker rm pgadmin
docker volume rm postgres_pgdata
docker compose -f compose.postgis.yml up
```

To verify if PostGIS is ready, in pgAdmin view, right click Database/pg4django, select `Query Tool`: 

```sql
select postgis_version();
```

It should output like this:

| postgis_version |
| :---: |
| 3.1 USE_GEOS=1 USE_PROJ=1 USE_STATS=1 |

If not, run `CREATE EXTENSION postgis;` in Query Tool.

The last step is to modify database setting in Django, then the Django project is equiped with PostGIS-ready PostgreSQL database.

```python
'ENGINE': 'django.contrib.gis.db.backends.postgis',
```

But it's not ready to run Django project, because GEOS, PROJ and GDAL should be installed prior to building PostGIS.[^8] Otherwise you will get error:

```shell
django.core.exceptions.ImproperlyConfigured: Could not find the GDAL library (tried "gdal", "GDAL", "gdal3.6.0", "gdal3.5.0", "gdal3.4.0", "gdal3.3.0", "gdal3.2.0", "gdal3.1.0", "gdal3.0.0", "gdal2.4.0", "gdal2.3.0", "gdal2.2.0"). Is GDAL installed? If it is, try setting GDAL_LIBRARY_PATH in your settings.
```

### Install Geospatial libraries

Refer to the documentation for installing geospatial libraries or just build Docker with the following dockerfile (saved in `dockerfile.base`).

```dockerfile
FROM python:3.10-slim-buster

RUN apt-get update \
    && apt-get install -y binutils libproj-dev gdal-bin \
    && apt-get install -y software-properties-common \
    && add-apt-repository ppa:ubuntugis/ppa \
    && apt-get install -y libgeos++-dev \
    && apt-get install -y proj-bin \
    && apt-get install -y gdal-bin \
    && apt-get install -y libgdal-dev
```

Build the base Docker image for our Django project and name it as `dj_base`:

```shell
docker build -f dockerfile.base -t dj_base .
```

And then change `dockerfile` as follows:

```dockerfile
FROM dj_base

WORKDIR /app

ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

COPY . .
RUN python -m pip install --upgrade pip && \
    pip3 install -r requirements.txt 
```

If you install all the Geospatial libraries in local environment and then run `python manage.py runserver` for development mode, you might get the error:


{{< alert danger >}}

OSError: /home/aaron/miniconda3/envs/dj/bin/../lib/libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by /lib/libgdal.so.30)

{{< /alert >}}

This can be solved by

```shell
ln -sf /usr/lib/x86_64-linux-gnu/libstdc++.so.6 ${CONDA_PREFIX}/lib
```

## Create Location Search Application

### Update the Model and Create Test Data

Modify the Django model `Restaurant` to include a `PointField` for storing the location coordinates, in order to do that, add `from django.contrib.gis.db import models` to replace `from django.db import models`. Add `lon` and `lat` fields to manually input geo location, and override `save` method to create a `Point` object to `locatoin` field whenever a save action is made.

```python
# from django.db import models
from django.contrib.gis.db import models
from django.contrib.gis.geos import Point

class Restaurant(models.Model):
    CHOICES_CUISINE_TYPE = [
        ('Asian food', 'Asian food'),
        ('European cuisine', 'European cuisine'),
        ('Cuisine of the Americas', 'Cuisine of the Americas'),
        ('African cuisine', 'African cuisine'),
    ]
    brand = models.CharField(verbose_name='Brand Name *', max_length=256)
    cuisine_type = models.CharField(max_length=128, choices=CHOICES_CUISINE_TYPE, null=True, blank=True)
    address = models.CharField(verbose_name='Address *', max_length=256)
    lon = models.FloatField(null=True, blank=True)
    lat = models.FloatField(null=True, blank=True)
    location = models.PointField(srid=4326, null=True, blank=True)

    def save(self, *args, **kwargs):        
        if self.lon and self.lat:
            self.location = Point(self.lon, self.lat, srid=4326)
        super().save(*args, **kwargs)
```

Note that the `srid=4326` parameter is necessary to specify the coordinate reference system (CRS) of the point. In this case, it corresponds to the WGS84 CRS commonly used for latitude and longitude coordinates.

Make sure to run the migrations to update your database schema after making this change.

And `lon` and `lat` to `fieldsets` in the `admin.py`:

```python
('Geo Information', {
    'fields': (
        ('lon', 'lat'),
    )
}),
```

Run the Django project, open Django admin `http://localhost:1024/admin/`, input some test data, such as:

| Brand Name | Address | Lon | Lat |
| --- | --- | --- | --- |
| Chick-fil-A	| 709 Yonge St, Toronto	| -79.4132781 | 43.6428802 | 
| La Vecchia Restaurant	| 90 Marine Parade Dr, Etobicoke | -79.6101167 | 43.6548902 | 
| Smash Kitchen	| 4261 Hwy 7, Unionville | -79.5045743 | 43.7589809 | 
| Windfield's Restaurant| 801 York Mills Rd, North York	| -79.3873467 | 43.7266729 | 
| JOEY | 75 O'Neill Rd, North York | -79.3693823 | 43.6888955
| Italian Kitchen | 200 Front St W Unit #G001, Toronto | -79.4080964 | 43.6448333 | 

### Create View for Location Search

To perform geo location search in our example, we can create a view for process user's request to search nearby restaurant within a certain distance. Suppose user input `lon`, `lat` and `distance`, we can utilise GeoDjango to do the geometric search. Add `search` function to `views.py`:

```python
import logging
from django.shortcuts import render
from django.contrib.gis.geos import Point
from django.contrib.gis.db.models.functions import Distance
from .models import Restaurant

logger = logging.getLogger(__name__)

def search(request):
    if request.method == 'GET':
        lon = request.GET.get('lon')
        lat = request.GET.get('lat')
        radius = request.GET.get('distance', 10000) # meters
        logger.debug(f'{lon=}, {lat=}, {radius=}')

        if lon and lat:
            input_location = Point(float(lon), float(lat), srid=4326)
            results = Restaurant.objects.annotate(
                distance=Distance('location', input_location)).filter(
                    distance__lte=radius).order_by('distance')
        else:
            results = Restaurant.objects.all()

        return render(request, 'search.html', {'results': results})
```

For our simple case, we just need to create a `Point` object from user's `lon` and `lat` and use the `annotate()` function to add a calculated field `distance` to each restaurant, representing the distance between the `location` field and the user's location. Then, the `filter()` function is used to retrieve the restaurants where the distance is less than or equal to the specified radius.

`search.html` is a Django template, put in the `djapp/templates/`:

```html
<form method="GET" action="{% url 'search' %}">
    <label for="lon">Longitude:</label>
    <input type="text" name="lon" id="lon" required>

    <label for="lat">Latitude:</label>
    <input type="text" name="lat" id="lat" required>

    <label for="distance">Distance (in meters):</label>
    <input type="number" name="distance" id="distance" value="5000">

    <button type="submit">Search</button>
</form>

<ul>
    {% for restaurant in results %}
        <li>
            <strong>{{ restaurant.brand }} - {{ restaurant.cuisine_type }}</strong><br>
            Address: {{ restaurant.address }}<br>
            Distance: {{ restaurant.distance.m|floatformat:0 }} meters
        </li>
    {% empty %}
        <li>No results found.</li>
    {% endfor %}
</ul>
```

Create `urls.py` in directory `djapp`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('search/', views.search, name='search'),
]
```

Append `path('djapp/', include('djapp.urls')),` to `urlpatterns`.

Run `docker compose up --build` to rebuild and run the Django project, input `http://localhost:1024/djapp/search` in the browser to open the location search application.

Input the following parameters:
- Longitude: -79.42
- Latitude: 43.65
- Distance (in meters): 10000

The print results:


{{< alert success >}}

**Chick-fil-A**  
Address: 709 Yonge St, Toronto  
Distance: 959 meters  

**Italian Kitchen**  
Address: 200 Front St W Unit #G001, Toronto  
Distance: 1117 meters  
  
**JOEY**  
Address: 75 O'Neill Rd, North York  
Distance: 5940 meters  
  
**Windfield's Restaurant**  
Address: 801 York Mills Rd, North York  
Distance: 8921 meters  

{{< /alert >}}

## Conclusion

In this example, we explore the powerful capabilities of GeoDjango and PostGIS for advanced location searches within Django applications. While this is a simple demonstration, we can enhance it further by optimizing queries using Google Maps API for address search and developing a RESTful API with Django REST Framework (DRF) to provide location search services. The code repository is available on GitHub.