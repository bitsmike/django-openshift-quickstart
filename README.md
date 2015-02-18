Django 1.6 on OpenShift
=======================

En primer lugar decir qué es Openshift:
“OpenShift es un producto de computación en la nube de plataforma como servicio de Red Hat.
Este software funciona como un servicio que es de código abierto bajo el nombre de “OpenShift Origin”, y está disponible en GitHub.
Los desarrolladores pueden usar Git para desplegar sus aplicaciones Web en los diferentes lenguajes de la plataforma.
OpenShift también soporta programas binarios que sean aplicaciones Web, con tal de que se puedan ejecutar en RHEL Linux. Esto permite el uso de lenguajes arbitrarios y frameworks.
OpenShift se encarga de mantener los servicios subyacentes a la aplicación y la escalabilidad de la aplicación como se necesite.” wikipedia


A continuación, se exponen los pasos a seguir para utilizar Django Rest Framework en Openshift

1/ Instalar las OpenShift Client Tools (rhc).

Algún comando de interes que podrá ser útil en el futuro:
– Para configurarlo se debe ejecutar: rhc setup. Nos pedirá login y password de openshift y nos permite crear una clave pública para luego conectar por ssh.
– Conectar ssh a tu aplicación: rhc ssh < app-name>
– Una vez conectado podríamos reiniciar la aplicación con: ctl_app restart
– Hacer log de la aplicación: rhc tail -a < app-name>



2/ Crear una aplicación Django que corra con python 2.7. Una de las formas de hacerlo es mediante el comando:

rhc create-app < app-name> python-2.7 --from-code 
               git://github.com/rancavil/django-openshift-quickstart.git
               
               
Si todo va correctamente en la consola te saldrá algo similar a esto:
URL: http://< app-name>-< usuario-openshift>.rhcloud.com/
SSH to: abcabcabcabcabcabc@< app-name>-< usuario-openshift>.rhcloud.com
Git remote: ssh://547b176b5973ca04c6000139@< app-name>-< usuario-openshift>.rhcloud.com/~/git/< app-name>.git/
Cloned to: C:/prueba/< app-name>

< app-name> = nombre deseado para tu aplicación.
< usuario-openshift> = tu usuario en openshift

Como nota decir, que me ha creado una copia de la aplicación creada en C:/prueba/. Esto es porque he ejecutado el create desde dentro del directorio c:/prueba/



3/ Instalar un cliente GIT. En mi caso, para windows utilicé Git Bash. Luego los cambios se subirán así:

?
1
2
3
git add .
git commit -m 'My changes'
git push


??????????????????????????????????????????????
==============================================
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

This git repository helps you get up and running quickly w/ a Django 1.6
installation on OpenShift.  The Django project name used in this repo
is 'openshift' but you can feel free to change it.  Right now the
backend is sqlite3 and the database runtime is found in
`$OPENSHIFT_DATA_DIR/db.sqlite3`.

Before you push this app for the first time, you will need to change
the [Django admin password](#admin-user-name-and-password).
Then, when you first push this
application to the cloud instance, the sqlite database is copied from
`wsgi/openshift/db.sqlite3` to $OPENSHIFT_DATA_DIR/ with your newly 
changed login credentials. Other than the password change, this is the 
stock database that is created when `python manage.py syncdb` is run with
only the admin app installed.

On subsequent pushes, a `python manage.py syncdb` is executed to make
sure that any models you added are created in the DB.  If you do
anything that requires an alter table, you could add the alter
statements in `GIT_ROOT/.openshift/action_hooks/alter.sql` and then use
`GIT_ROOT/.openshift/action_hooks/deploy` to execute that script (make
sure to back up your database w/ `rhc app snapshot save` first :) )

With this you can install Django 1.6 on OpenShift.

Running on OpenShift
--------------------

Create an account at http://openshift.redhat.com/

Install the RHC client tools if you have not already done so:
    
    sudo gem install rhc

Create a python-2.7 application

    rhc app create -a djangoproj -t python-2.7

Add this upstream repo

    cd djangoproj
    git remote add upstream -m master git://github.com/rancavil/django-openshift-quickstart.git
    git pull -s recursive -X theirs upstream master

####Note:
If you want to use the Redis-Cloud with Django see [the wiki](https://github.com/rancavil/django-openshift-quickstart/wiki/Django-1.6-with-Redis-Cloud) 

Then push the repo upstream

    git push

Here, the [admin user name and password will be displayed](#admin-user-name-and-password), so pay
special attention.
	
That's it. You can now checkout your application at:

    http://djangoproj-$yournamespace.rhcloud.com

Admin user name and password
----------------------------
As the `git push` output scrolls by, keep an eye out for a
line of output that starts with `Django application credentials: `. This line
contains the generated admin password that you will need to begin
administering your Django app. This is the only time the password
will be displayed, so be sure to save it somewhere. You might want 
to pipe the output of the git push to a text file so you can grep for
the password later.

When you make:

     git push

In the console output, you must find something like this:

     remote: Django application credentials:
     remote: 	user: admin
     remote: 	SY1ScjQGb2qb

Or you can go to SSH console, and check the CREDENTIALS file located 
in $OPENSHIFT_DATA_DIR.

     cd $OPENSHIFT_DATA_DIR
     vi CREDENTIALS

You should see the output:

     Django application credentials:
     		 user: admin
     		 SY1ScjQGb2qb

After, you can change the password in the Django admin console.

Django project directory structure
----------------------------------

     djangoproj/
        .gitignore
     	.openshift/
     		README.md
     		action_hooks/  (Scripts for deploy the application)
     			build
     			post_deploy
     			pre_build
     			deploy
     			secure_db.py
     		cron/
     		markers/
     	setup.py   (Setup file with de dependencies and required libs)
     	README.md
     	libs/   (Adicional libraries)
     	data/	(For not-externally exposed wsgi code)
     	wsgi/	(Externally exposed wsgi goes)
     		application (Script to execute the application on wsgi)
     		openshift/	(Django project directory)
     			__init__.py
     			manage.py
     			openshiftlibs.py
     			settings.py
     			urls.py
     			views.py
     			wsgi.py
     			templates/
     				home/
     					home.html (Default home page, change it)
     		static/	(Public static content gets served here)
     			README

From HERE you can start with your own application.
