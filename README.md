# Integrantes
Juan Ramírez Ramírez
Jian Carlo Gallego

# Introducción
A continuación se expone el desarrolloro de una api cliente-servidor hecha en python que permite realizar peticiones GET, HEAD y POST en la versión 1.1 de HTTP

# Configuración

En caso de bajarse el proyecto local, se debe correr en la rama "runlucal" puesto que en esta rama se le solicita al usuario por la terminal el puerto, la ruta del logfile (log.txt) y la ruta de las imágenes (Paginas/)

Comando para correr el proyecto local:
Linux: python3 Servidor.py 1234 log.txt Paginas/
Windows: python Servidor.py 1234 log.txt Paginas/
Importante estar ubicado en la raíz

El proyecto se desplegó en amazon en la siguiente ip y puerto: http://3.144.3.31:8501/, aquí se pueden hacer pruebas, importante estar en la rama master 
Ejemplo de la petición en AWS: http://3.144.3.31:8501/Paginas/Caso4.html

![alt text](https://assets.hibot.us/images/dev-content-based/666f5f0bad467fde1066e60a06766f9c463760cc9494614a57897db6f25e7e64@jpg)

![alt text](https://assets.hibot.us/images/dev-content-based/a35090cd6c753f31d29816bf026a9ead3e134b8b8eaabb1f210855d16feca6fa@jpg)



# Desarrollo

Aquí podremos ver cómo funciona el código con su respectiva documentación en cada método.

```python
import socket
import sys
import os
import time
from threading import Thread

#Argumentos de entrada $./server <HTTP PORT> <Log File> <DocumentRootFolder>
puerto = int(sys.argv[1])
log_file = sys.argv[2]
document_root = sys.argv[3]

#Archivo Log que voy a crear
log = open(log_file, 'w')

# Creamos un objeto socket para la conexión
servidor = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Definimos la dirección IP y el puerto del servidor
ip_servidor = '127.0.0.1'
puerto_servidor = puerto

# Enlazamos el socket al servidor
servidor.bind((ip_servidor, puerto_servidor))

# Ponemos el servidor en modo escucha
servidor.listen()

def manejar_solicitud(cliente):
    log = open(log_file, 'a')

    # Recibimos la solicitud del cliente
    solicitud = cliente.recv(1024).decode()
    log.write(solicitud)

    # Extraemos la ruta solicitada de la solicitud
    ruta_archivo = solicitud.split()[1]

    # Si la ruta es '/', servimos la página de inicio
    if ruta_archivo == '/':
        ruta_archivo = '/' + document_root + '/Caso1.html'
    elif ruta_archivo == '/Caso2':
        ruta_archivo = '/' + document_root + '/Caso2.html'
    elif ruta_archivo == '/Caso3':
        ruta_archivo = '/' + document_root + '/Caso3.html'
    elif ruta_archivo == '/Caso4':
        ruta_archivo = '/' + document_root + '/Caso4.html'
        
    # Manejamos la solicitud en función del método HTTP
    metodo = solicitud.split()[0]

    if metodo == 'GET':
        # Intentamos abrir el archivo solicitado
        try:
            print(os.getcwd() + ruta_archivo)
            with open(os.getcwd() + ruta_archivo, 'rb') as archivo:
                contenido_archivo = archivo.read()
                codigo_respuesta = 'HTTP/1.1 200 OK\n'
                tipo_contenido = 'text/html'
        except FileNotFoundError:
            contenido_archivo = b'<html><body><h1>Error 404: Archivo no encontrado</h1></body></html>'
            codigo_respuesta = 'HTTP/1.1 404 Not Found\n'
            tipo_contenido = 'text/html'

        # Enviamos la respuesta al cliente
        encabezado_respuesta = codigo_respuesta + 'Content-Type: ' + tipo_contenido + '\n\n'
        respuesta = encabezado_respuesta.encode() + contenido_archivo
        cliente.sendall(respuesta)

    elif metodo == 'POST':
        # Recibimos los datos del formulario
        datos_formulario = solicitud.split('\r\n\r\n')[1]
        print(f'Datos del formulario: {datos_formulario}')

        # Enviamos la respuesta al cliente
        codigo_respuesta = 'HTTP/1.1 200 OK\n'
        tipo_contenido = 'text/html'
        contenido_respuesta = '<html><body><h1>Gracias por enviar el formulario!</h1></body></html>'.encode()
        encabezado_respuesta = codigo_respuesta + 'Content-Type: ' + tipo_contenido + '\n\n'
        respuesta = encabezado_respuesta.encode() + contenido_respuesta
        cliente.sendall(respuesta)

    elif metodo == 'HEAD':
        # Verificamos si el archivo solicitado existe
        if os.path.isfile(os.getcwd() + ruta_archivo):
            codigo_respuesta = 'HTTP/1.1 200 OK\n'
        else:
            codigo_respuesta = 'HTTP/1.1 404 Not Found\n'

        # Enviamos la respuesta al cliente
        encabezado_respuesta = codigo_respuesta + '\n'
        respuesta = encabezado_respuesta.encode()
        cliente.sendall(respuesta)

    elif metodo not in ['GET', 'POST', 'HEAD']:
        codigo_respuesta = 'HTTP/1.1 400 La petición solicitada por el cliente no pudo ser procesada.\n'
        tipo_contenido = 'text/html'
        contenido_respuesta = '<html><body><h1>Error 400: Solicitud incorrecta</h1></body></html>'.encode()
        encabezado_respuesta = codigo_respuesta + 'Content-Type: ' + tipo_contenido + '\n\n'
        respuesta = encabezado_respuesta.encode() + contenido_respuesta
        cliente.sendall(respuesta)

    # Cerramos la conexión con el cliente
    log.close()
    cliente.close()

# Esperamos conexiones de clientes
while True:
    print('Esperando conexiones...')
    cliente, direccion = servidor.accept()
    cliente.settimeout(10) # Tiempo de espera de 10 segundos
    print(f'Conexión establecida desde {direccion[0]}:{direccion[1]}')
    Thread(target=manejar_solicitud, args=(cliente,)).start()

```

## Pruebas

Para probar nuestro servidor se ejecutaron las peticiones mediante postman

GET a la página 1 con algunos hipertextos y una imagen.
![alt text](https://assets.hibot.us/images/dev-content-based/926444ab1a17dfb30f4bf48b7623feaedf6297676bf34c37dedbf4c9dbe17c82@jpg)

GET a la página 2 con algunos hipertextos y múltiples imágenes.
![alt text](https://assets.hibot.us/images/dev-content-based/f974b41cfc2475558422735b564488198063a34701c4ef155f66b1971e87b53b@jpg)

GET a la página 3 que contiene un solo archivo de aproximadamente un tamaño de 1MB.
![alt text](https://assets.hibot.us/images/dev-content-based/13baea0e29db3b01d8d2bb8fb401c3bfbb555a9928730c758da5fe6c660e89b0@jpg)

GET a la página 4 que contiene múltiples archivos y que aproximadamente tiene un tamaño de 1MB
![alt text](https://assets.hibot.us/images/dev-content-based/ddf6cba8e2a4cc11f9ca7d224f9045574855a4f3b3ee92b610717d2df276ef12@jpg)

GET a url desconocida
![alt text](https://assets.hibot.us/images/dev-content-based/2822c18e61f1801231d04dc38d9515ff3a69877c649bfdde4713bb5cb113c4e0@jpg)

POST a la url
![alt text](https://assets.hibot.us/images/dev-content-based/9972387a20ad55ec1b5eabe18a6c6b35d22662a66060743e19beefa60ed3ea56@jpg)

HEAD a la url
![alt text](https://assets.hibot.us/images/dev-content-based/df86c011a1059ece7cd1497d261cf43a015c2dd14b77d0fe7f1b445399030b07@jpg)




# Conclusiones

Podríamos concluir que nuestra aplicación es un servidor que permite el intercambio de información en el formato http y que este es utilizado por la mayoría de páginas web que hay en la internet. Además, de que su implementación en un lenguaje como python puede llegar a ser sencilla y óptima a la vez.

El protocolo http comprime en cierta medida el tamaño de los archivos, esto lo identificamos ya que las imágenes tienen las medidas aproximadas de los casos de prueba, pero a la hora de ver el size en postman, vemos que es mucho menor.

# Referencias

https://www.postman.com/
https://www.escribecodigo.com/sockets-en-python/