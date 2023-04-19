# Integrantes
Juan Ramírez Ramírez
Jian Carlo Gallego

# Introducción
A continuación se expone el desarrolloro de una api cliente-servidor hecha en python que permite realizar peticiones GET, HEAD y POST en la versión 1.1 de HTTP

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

# Conclusiones

# Referencias

https://www.postman.com/