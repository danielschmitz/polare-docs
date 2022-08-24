Instalação do Sistema
=====================

O sistema Polare e o banco de dados `Postgresql <https://www.postgresql.org/>`_ serão executados em contêiner
(Docker) conforme descrito em :hoverxref:`Pré-Requisitos de Instalação <pre_requisitos>`.

Criar as seguintes pastas:
    - polare-imagem
    - polare-db
    - jar
    - ambiente-polare

Criar um arquivo com o nome ``Dockerfile`` dentro da pasta ``polare-imagem`` e adicionar o seguinte conteúdo:

.. code-block:: docker

    FROM openjdk:17-alpine

    RUN mkdir -p /opt/polare/
    RUN apk update && apk upgrade

    EXPOSE 8080
    CMD ["java", "-jar", "/opt/polare/polare-frontend-thymeleaf.jar"]


Este é um arquivo para gerar uma imagem com o `Java <https://www.java.com>`_ na versão recomendada pela UFRN (OpenJDK-17). Para que o
Docker crie a imagem *polare-web*, conforme o especificado no ``Dockerfile``, deve-se executar o seguinte
comando dentro da pasta ``polare-imagem``:

.. code-block:: bash

    docker build -t polare-web .


Para criar o *.jar* do sistema Polare deve-se acessar a pasta onde o código fonte foi salvo e executar o
comando:

.. code-block:: bash

    ./mvnw spring-javaformat:apply clean install -P jar -B -Dmaven.test.skip=true -Dmaven.javadoc.skip=true -Ddependency-check.skip=true


.. note::
    Para obter o código fonte do sistema Polare é necessário entrar em contato com a UFRN para maiores informações.


O *.jar* será salvo na pasta ``polare-frontend-thymeleaf/target``, com o nome:
``polare-frontend-thymeleaf.jar``. Este arquivo deve ser enviado para a pasta ``jar`` (criada anteriormente).

Para configurar e iniciar os contêiners adequadamente deve-se criar os arquivos ``nginx.conf`` e
``docker-compose.yml`` na pasta ``ambiente-polare`` (criada anteriormente) com os seguintes conteúdos:

**nginx.conf**

.. code-block:: nginx

  proxy_cache_path /tmp/NGINX_treinamento_cache/ keys_zone=backcache:10m;

  upstream polare {
    ip_hash;
    server 192.168.0.191:8080; # ajusar o ip do server
  }

  server {
    listen 80;
    server_name polare.edu.br; # ajustar o server_name

    client_max_body_size 128M;

    access_log /var/log/nginx/polare-access.log;
    error_log /var/log/nginx/polare-error.log;

    # Redirect all HTTP to HTTPS
    return 301 https://polare.edu.br; # ajustar o server_name
  }

  server {
    listen 443 ssl http2;
    server_name polare.edu.br; # ajustar o server_name

    client_max_body_size 128M;

    access_log /var/log/nginx/polare-access.log;
    error_log /var/log/nginx/polare-error.log;

    ssl_certificate      /etc/nginx/ssl/cert.bundle; # ajustar certificado
    ssl_certificate_key  /etc/nginx/ssl/cert.key; # ajustar certificado

    ssl_session_cache   shared:SSL:1m;
    ssl_prefer_server_ciphers  on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    location = / {
      return 302 /polare;
    }

    location / {
      proxy_pass http://polare/; # upstream
      proxy_cache backcache;
      proxy_set_header X-Real-IP  $remote_addr;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header Host $host;
      proxy_set_header X-Real-Port $server_port;
      proxy_set_header X-Real-Scheme $scheme;
    }

    location /polare {
      proxy_pass http://polare/polare;
    }
  }

**docker-compose.yml**

.. code-block:: docker

  version: "3.3"
  services:
    nginx:
      image: nginx # ajustar caso seja utilizada uma imagem customizada para o nginx
      hostname: nginx
      restart: unless-stopped
      ports:
        - "80:80"
        - "443:443"
      environment:
        - TZ=America/Belem
      volumes:
        - type: bind
          source: ./nginx.conf
          target: /etc/nginx/conf.d/default.conf
          read_only: true
      networks:
        - rede
      depends_on:
        - polare-db
        - polare-web

    polare-db:
      container_name: polare-db
      image: postgres:12
      hostname: polare-db
      ports:
        - "5432:5432"
      environment:
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
      healthcheck:
        test: [ "CMD-SHELL", "pg_isready -U postgres" ]
        interval: 10s
        timeout: 5s
        retries: 5
      restart: unless-stopped
      volumes:
        - /home/administrador/polare-db:/var/lib/postgresql/data
      networks:
        - rede
    polare-web:
      image: polare-web
      container_name: polare-web
      hostname: polare-web
      restart: unless-stopped
      ports:
        - "8080:8080"
      volumes:
        - type: bind
          source: /home/administrador/jar/polare-frontend-thymeleaf.jar
          target: /opt/polare/polare-frontend-thymeleaf.jar
      networks:
        - rede
  volumes:
    polare_db:
  networks:
    rede:
      driver: bridge


O seguinte comando deve ser executado na pasta ``ambiente-polare`` para levantar os contêiners (incluindo o
sistema Polare):

.. code-block:: bash
    
    docker-compose up -d


.. figure:: /_static/img/login-polare.png
    :align: center

    Tela de login do sistema Polare


.. warning::

    O login no sistema Polare só poderá ser efetuado se uma infraestrutura SSO estiver instalada e configurada.
