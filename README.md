# Prosoul [![Build Status](https://travis-ci.org/Bitergia/prosoul.svg?branch=master)](https://travis-ci.org/Bitergia/prosoul)

**Prosoul** is a software quality models manager to create, import/export, view and edit
models. It has been implemented in the context of the EU funded research project [CROSSMINER](https://www.crossminer.org/).


## Description

The goal of this project is to create a web editor and viewer for [software quality models](https://github.com/borisbaldassari/se-quality-models) that are used to show metrics in a meaningful way. Also, importers and exporters for the different quality models used in the industry will be implemented so they can be used as models or evolved for the creation of new quality models.

Based on the models, visualizations and assessment for the projects are generated. In the initial version, the metrics supported will be the [CROSSMINER ones](https://github.com/crossminer/crossminer/tree/dev/web-dashboards/ossmeter-metrics) and the Kibana dashboards will be created using [GrimoireLab](http://grimoirelab.github.io/).

The final goal is to have a tool for "Automatic Project Assessment and Visualization based on Evolved Quality Models".

## Architecture

![arch-diag](doc/prosoul-arch.png?raw=true)

> **NOTE**: The original name for this project was **Meditor** but it was changed to **Prosoul** because meditor already was used in pip.


## Installation and Set up

**Prosoul** is a Django application. The recommended way to execute it is inside a python virtual environment. 

You will need Elasticsearch and Kibana in order to execute Prosoul. You can install them using docker-compose. You can find a walkthrough here, [Getting the containers](https://github.com/chaoss/grimoirelab-sirmordred/blob/master/Getting-Started.md#getting-the-containers-). You can omit (comment/remove) the mariadb section.

### Requirements

* Python >=3.4
* setuptools
* django
* kidash
* requests
* python3-tk
* matplotlib

You can run the application by three ways, [source code](#from-source-code), [pip package](#from-pip-package) and [docker](#from-docker).

### from source code

This is the recommended way if you want to contribute to the development of ProSoul.

```
$ git clone https://github.com/Bitergia/prosoul.git
$ cd prosoul/django-prosoul
prosoul/django-prosoul $ mkdir -p VENV_DIR
prosoul/django-prosoul $ python3 -m venv VENV_DIR
prosoul/django-prosoul $ source VENV_DIR/bin/activate
prosoul/django-prosoul (VENV_DIR) $ pip3 install -r requirements.txt
```

And to start the Django application:

```
prosoul/django-prosoul (VENV_DIR) $ python3 manage.py makemigrations
prosoul/django-prosoul (VENV_DIR) $ python3 manage.py migrate
prosoul/django-prosoul (VENV_DIR) $ python3 manage.py runserver
```

By default, the application will be accessible in: http://127.0.0.1:8000/

There is a demo video in YouTube about how to install the Prosoul application from the source code.

**Quick Links**

1. Setting up Prosoul with PyCharm - [0:20](https://www.youtube.com/watch?v=-wU1ck4ZrUw&t=20s)
2. Setting up Prosoul with Terminal - [3:33](https://www.youtube.com/watch?v=-wU1ck4ZrUw&t=213s)

### from pip package

There is also a pip package with the Django application. You can just deploy it in a Django site:

```
$ mkdir -p VENV_DIR
$ python3 -m venv VENV_DIR
$ source VENV_DIR/bin/activate
(VENV_DIR) $ pip install django
(VENV_DIR) $ pip install django-prosoul
```

and you need to add the application following the normal steps in Django:

```
(VENV_DIR) $ django-admin startproject mysite
(VENV_DIR) $ cd mysite

mysite (VENV_DIR) $ vi mysite/settings.py
...
INSTALLED_APPS = [
    'django.contrib.admin',
	...
	'prosoul'
	]

mysite (VENV_DIR) $ vi mysite/urls.py
...
from django.conf.urls import include, url
...
urlpatterns = [
    path('admin/', admin.site.urls),
    path('prosoul/', include('prosoul.urls')),
]

mysite (VENV_DIR) $ python manage.py migrate
mysite (VENV_DIR) $ python manage.py runserver
```

### from docker

A docker image is also available to execute the application. 

> NOTE: The [docker-compose.yml](docker/docker-compose.yml) has the elasticsearch and kibiter configurations too 
along with prosoul.

```
prosoul/docker $ docker-compose up
```

The applications would be running on the respective allocated ports.


## How to

This section explains how to create a quality model (QM), view it, visualize it in Kibana and assess it on a set of 
projects.
A QM is identified by a title and composed of several `Goals`, which aim at measuring specific aspects of your projects, 
such as `Activity`, `Community` and `Processes`. Each goal is defined by a set of `Attributes`, they aim at characterizing
a given goal. For instance, the `Activity` goal could include `Code` and `Bugs`, while `Community` could have `Coders` and
`Reporters` as attributes. Each attribute is then mapped to one or more `Metrics`. A `Metric` has a name, a 5-level 
threshold (used to rate the project wrt a metric) and the data implementing the metric. In the case of CROSSMINER, the 
metric implementation is the `metric_name` field of the metrics extracted from SCAVA and stored in the 
index `scava-metrics`. For example, the attribute `Code` previously created could be mapped on the SCAVA metric 
`Patches`, while the attribute `Bugs` could be represented by the SCAVA metrics `Bugs`, `Won't-Fix Bugs` and `Fixed Bugs`.

In the following sections, the various functions of the application are described.

### Edit

The Edit function allows to create (modify and import) a QM. 

To create a QM, click on the first `Add` button and set a name for your QM (e.g., `MyFirstQM`) in the window that popped up.
Then, Prosoul will automatically open a chain of windows to to let you insert your first `Goal`, its `Attributes` and the 
corresponding `Metrics`. For each metric, you will have to map it to an existing metric in your data. In the case of 
CROSSMINER, this is done via the `metric_name` attribute stored
in the index `scava-metrics`. 

### View

The View function allows you to view the QM in the form of a tree.

Select the QM which you would like to view in the application and you can see the tree view of the QM. You can also view 
some of the information like the number ofAttributes and Metrics in the selected QM. Click on any node to know more details 
about the attribute.

### Visualize

The Visualize function allows to materialize your QM in Kibana through dashboards, which are created based on templates 
available [here](https://github.com/Bitergia/prosoul/tree/master/django-prosoul/prosoul/panels/templates).

In the form, you have just to select the target QM, the ElasticSearch and Kibana URLs, the ElasticSearch index 
where the data is stored, the attribute template and the metrics data backend (which tells Prosoul how to process the 
data in the index). In the case of CROSSMINER, all form fields except the `Quality Model` one are already set to be used
with the default Docker configuration of the [scava-deployment](https://github.com/crossminer/scava-deployment/tree/dev#scava-deployment).

Once the form is filled, by clicking on the `Create` button, a set of dashboards (one per each attribute) are uploaded
to Kibana and available in the menu `Dashboard`. 

### Assess

The Assess function allows to perform an assessment of the QM over the projects included in your data.

In the form, you have just to select the target QM, the ElasticSearch URL, the index where the data is stored and
the metrics data backend. In the case of CROSSMINER, all form fields except the `Quality Model` one are already set to be 
used with the default Docker configuration of the [scava-deployment](https://github.com/crossminer/scava-deployment/tree/dev#scava-deployment).

Once the form is filled, by clicking on the `Create` button, you will be redirected to a page summarizing how the projects
in your data comply with the QM selected.

### Import / Export

Import and export for quality models can be done using the web interface or from the command line:

* Import a model from a OSSMeter JSON file:
```
prosoul/django-prosoul $ PYTHONPATH=. prosoul/prosoul_import.py -f prosoul/data/ossmeter_qm.json --format ossmeter
```

* Export a model to a GrimoireLab JSON file:
```
prosoul/django-prosoul $ PYTHONPATH=. prosoul/prosoul_export.py -f prosoul/data/ossmeter_qm_grimoirelab.json --format grimoirelab -m "Default OSSMETER quality model"
```


## Tutorials

There is a demo video about how to install the Prosoul application from the source code. 

<div align="center">

[![Setting up Prosoul](https://img.youtube.com/vi/-wU1ck4ZrUw/0.jpg "Setting up Prosoul | YouTube")](https://www.youtube.com/watch?v=-wU1ck4ZrUw)

</div>

There are two introductory videos: showing the import and view feature, and how to use the editor for adding a new quality model:

* Import and view of quality models: [prosoul-intro.webm](https://raw.githubusercontent.com/Bitergia/prosoul/master/doc/meditor-intro.webm)
* Adding a new quality model: [prosoul-editor.webm](https://raw.githubusercontent.com/Bitergia/prosoul/master/doc/meditor-editor.webm)
* Creating a viz and an assessment based on a quality model: [prosoul-viz-assess.webm](https://raw.githubusercontent.com/Bitergia/prosoul/master/doc/meditor-viz-assess.webm)


## Source code and contributions

All the source code is available in the [Prosoul GitHub repository](https://github.com/Bitergia/prosoul). Please, open an issue if you want to report a bug, ask for a new feature, or just comment something or send pull requests if you have proposals to change the source code.

The [code of conduct](CODE_OF_CONDUCT.md) must be followed in all contributions to the project.


## License

Licensed under GNU General Public License (GPL), version 3 or later.
