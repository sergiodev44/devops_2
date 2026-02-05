# Leer 
- Esta aplicación de pruebas que combina docker, github actions y EC2 de AWS, usando una aplicación sencialla llamada galeria.

- Esta aplicación utiliza secretos para acceder a dockerhub y a la instacia EC2.

- El workflow del actions debe esta en la carpeta .github/workflows

- El repo debe de tener la branch rule de forzar el pull request antes del merge a main para triggerear las actions.

## Actions

- EL primer job crea my imagen docker, con el checkuot, en el runner the github, se logea y hace un push a mi cuenta sergioo7777.

- El job de aws, lanza mi imagen docker en una instancia EC2. Crea la instacia ubuntu, copia el docker compose via ssh key con el scp y mis credenciales. Después de 60 segundo hace un docker compose down, por si acaso, y después levanta la imagen.     

## Comandos Principales

1. Creo un entorno virtual para correr la aplicación "galeria" de flask en local
    - $ python3 -m venv venv
    - $ source venv/bin/activate
    - (venv) $ pip install -r requirements.txt 
    - (venv) $ python src/app.py 

- La pruebo con firefox --window-private localhost:5000 & en incógnito para que no haya problemas con la caché.

- Creo la imagen docker
    - (venv) $ docker build -t sergioo7777/galeria:latest .

- La lanzo y la pruebo como antes
    - (venv) $ docker run -d -p 80:5000  --name testgaleria sergioo7777/galeria:latest
- Finalmente la corro con el docker compose 
    - (venv) $ docker compose  up -d 
    - (venv) $ firefox http://localhost:80 &
    - (venv) $ docker compose  down
- La subo a mi docker hub
    - (venv) $ docker push  sergioo7777/galeria:latest

2. Ahora para la parte de la actions, me creo el creo el token  para hacer el dockerhub push con privilegios leer escribir y borrar
- En github repo, voy a seguridad y a secretos clásicos y añado un secreto al repo DOCKERHUN_TOKEN y DOCKERHUB_USERNAME que uso después en los actions
- Uso el script de action del profesor en el archivo docker-ci-cd.yml en la carpeta .github/workflows en el root de mi repo para que github lo auto conecta al hacer el trigger del action. En nuestro caso cada vez que hago un push, que solicta una pull request. El push debe contener cambios en src/ , la carpeta principal de la app para que se active al action.

- CI/CD se refiere a integración y despliegue continuo (gracias al uso de secretos), el marketplace de guthub proporciona muchos más tipos de actions

- Para probar el action hago:
    - checkout -b rama_pruebas
    - hago un cambio en src/app.py por ejemplo
    - git add src
    - git commit -m "cambio prueba"
    - git push -u origin rama_pruebas

- Al entrar en github me sugiere hacer un pull request porque main está behind. La acepto y entonces el action empieza a correr.
- Naturalmente pasa el test. 
- El action en cuestión hace una serie de jobs (trabajos):
    - El job se llama build (construir)
    - el job corre en un entorno virtual de Ubuntu que proporciona github
    - actios/checkout se descarga el repo en el runner
    - docker/login-action usa los secretos que creamos antes para permitir al runner the github actions autenticarse en dockerhub.
    - docker/build-push-actions crea la image docker usando como cntexto el directorio actual y hace un push de la nueva nueva imagen.

3. Para el despliegue en AWS, creo una instacia EC2 con la plantilla de clase corriendo debian con docker instalado. 
- En github secrets añado los secretos de username = admin para mi caso, hotsname = la ip pública de mi máquina y el privatekey = que para mi es el labuser.pem ( el contenido de ese archivo).

- finalmente en nuestra archivo .yml, debajo del job build, añado el job "aws".

- tiene needs: build, lo que significa que hasta que build no esté "OK", este job no empieza,
- de forma similar el runner corre sobre un entorno virtual de ubuntu
- tenemos otro actions/checkout que hace lo mismo que antes porque es un job nuevo, clona el repo para que lo use el runner de github
- appleboy/scp-action hace una copia de docker compose desde la máquina a la instacia de EC2 via SSH. Basicamente copia el docker-compose.yml en la carpeta /home/admin de la instacia EC2.

- appleyboy/ssh-action se logea en mi instancia EC2 usando los secretos y corre una serie de comandos en la shell:
    - primero para asegurar que dockerhub reconoce la nueva versión de la imagen que hemos hecho un push
    - hacemos un docker compose down --rmi all
        - para para todos los posible contenedores corriendo, los elimina y sus imagenes
    - ahora entonces si hacemos el docker compose up -d

- Para probarlo hago lo mismo:
    - checkout -b rama_pruebas_aws
    - hago un cambio en src/app.py por ejemplo
    - git add src
    - git commit -m "cambio prueba aws"
    - git push -u origin rama_pruebas_aws

- En el mismo push me podrá el link del pull request, la creo a main y arranca el action.

- El action functiona perfectamente primero hace el build y después tras pasar el test el despliegue en AWS y todo funciona perfectamente. 

    

