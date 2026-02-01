# Leer 
- Esta aplicación de pruebas que combina docker, github actions y EC2 de AWS, usando una aplicación sencialla llamada galeria.

- Esta aplicación utiliza secretos para acceder a dockerhub y a la instacia EC2.

- El workflow del actions debe esta en la carpeta .github/workflows

- El repo debe de tener la branch rule de forzar el pull request antes del merge a main para triggerear las actions.

## Actions

- EL primer job crea my imagen docker, con el checkuot, en el runner the github, se logea y hace un push a mi cuenta sergioo7777.

- El job de aws, lanza mi imagen docker en una instancia EC2. Crea la instacia ubuntu, copia el docker compose via ssh key con el scp y mis credenciales. Después de 60 segundo hace un docker compose down, por si acaso, y después levanta la imagen.     