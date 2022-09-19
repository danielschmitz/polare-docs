Instalação do Sistema
=====================

O sistema Polare e o banco de dados `Postgresql <https://www.postgresql.org/>`_ serão executados em contêiner
(Docker) conforme descrito em :hoverxref:`Pré-Requisitos de Instalação <pre_requisitos>`.

No momento que esta documentação é escrita, o sistema Polare precisa ser distribuído em um servidor Java
*web*. Neste guia utilizaremos o `Tomcat <https://tomcat.apache.org/>`_ versão 9.

O procedimento de implantação do sistema Polare consiste em criar uma imagem Tomcat 9 e em seguida executar
essa imagem via Docker. Essa imagem deverá conter as configurações locais da instituição (conexões com o banco
de dados, senhas, mapeamento das rotas da infraestrutura de API, etc.) e o sistema Polare (arquivos ``.war``
construídos utilizando a ferramenta Maven a partir do código fonte).

Construção dos artefatos .war (Polare)
--------------------------------------

O procedimento para gerar os arquivos ``.war`` consiste em clonar o código fonte do sistema Polare via Git e
executar o Maven para construir esses arquivos ``.war``.

**Clonar o código fonte.** O seguinte comando deve ser executado a partir de um diretório na linha de comando
para clonar o código fonte do sistema Polare (é necessário informar as suas credenciais de acesso):

.. code-block:: bash

    git clone -b develop https://gitdesenvolvimento.info.ufrn.br/dev/polare.git


.. note::

    Um diretório chamado polare será criado contendo todo o código fonte do sistema. A flag ``-b develop``
    clona o código fonte já na *branch* ``develop``.


**Construir os arquivos .war utilizando Maven.** O seguinte comando deve ser executado a partir do diretório
polare (que contém o código fonte do sistema):

.. code-block:: bash

    ./mvnw clean install -Pwar -B -Dmaven.test.skip=true -Dmaven.javadoc.skip=true -Ddependency-check.skip=true


A ferramenta Maven fará o *download* de todas as dependências necessárias para o projeto finalizando com a
construção dos arquivos ``.war``.

Caso ocorra o erro:

.. code-block:: bash

  Error: Could not find or load main class org.apache.maven.wrapper.MavenWrapperMain
  Caused by: java.lang.ClassNotFoundException: org.apache.maven.wrapper.MavenWrapperMain


É necessário executar o comando seguinte para configurar o ``maven wrapper``:

.. code-block:: bash

  mvn -N io.takari:maven:wrapper


Dois arquivos ``.war`` serão gerados:

    1. polare/polare-frontend-publico/target/polare-frontend-publico.war
    2. polare/polare-frontend-restrito/target/polare-frontend-restrito.war


Criação da Imagem Tomcat
------------------------

Crie um diretório contendo os seguintes arquivos:

    1. apache-tomcat-9.0.65 (diretório)
    2. copiar-ears-e-iniciar-tomcat.sh
    3. Dockerfile


O conteúdo desses arquivos é descrito a seguir.

**apache-tomcat-9.0.65**

Esse é o diretório contendo o Tomcat 9 descompactado que pode ser obtido no link
`https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.zip
<https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.zip>`_


**copiar-ears-e-iniciar-tomcat.sh**

.. code-block:: bash

    #!/bin/sh
    cp /polare/deploy/* /opt/tomcat/webapps/
    /opt/tomcat/bin/catalina.sh run


**Dockerfile**

.. code-block:: docker

    FROM openjdk:17-alpine

    RUN mkdir -p /opt/tomcat/
    RUN mkdir -p /polare/deploy

    COPY ./apache-tomcat-9.0.65 /opt/tomcat
    COPY ./copiar-ears-e-iniciar-tomcat.sh /polare/

    EXPOSE 8080 8443 8009 9999 8787
    env CATALINA_HOME /opt/tomcat

    ENTRYPOINT ["/polare/copiar-ears-e-iniciar-tomcat.sh"]


Os arquivos 1 e 2 serão inseridos na imagem que será criada através do Dockerfile. Neste guia o nome da imagem
Tomcat será tomcat-9. Para criar a imagem execute o comando a seguir a partir do diretório contendo os arquivos
descritos anteriormente:

.. code-block:: bash

    docker build -t tomcat-9 .


.. note:: O nome da imagem (tomcat-9) é referenciado no arquivo docker-compose.yml


Execução do Ambiente
--------------------

Crie um diretório contendo os seguintes arquivos:

    1. wars (diretório)
    2. docker-compose.yml
    3. nginx.conf
    4. catalina.sh

O conteúdo desses arquivos é descrito a seguir.

**wars (diretório).** É necessário copiar os arquivos ``.war`` gerados anteriormente para este diretório.
Nesta implantação os arquivos ``.war`` foram renomeados para polare.war (referente ao arquivo
polare-frontend-restrito.war) e polare-publico.war (referente ao arquivo polare-frontend-publico.war).

.. warning::

    O Tomcat atribui o caminho do contexto da aplicação em função do nome do arquivo ``.war`` contido no
    diretório ``apache-tomcat-9.0.65/webapps/`` por padrão. Por exemplo, um arquivo (aplicação) chamado
    ``polare.war``, pode ser acessado via http://localhost:8080/polare.


**docker-compose.yml.** É necessário fazer ajustes nesse arquivo em função das configurações do ambiente
local (as linhas com o comentário ``# ALTERAR`` devem ser modificadas).

.. code-block:: docker

    version: "3.2"
    services:
      nginx:
        image: nginx
        hostname: nginx
        restart: unless-stopped
        ports:
          - "80:80"
          - "443:443"
        environment:
          - TZ=America/Belem # ALTERAR
        volumes:
          - type: bind
            source: ./nginx.conf
            target: /etc/nginx/conf.d/default.conf
            read_only: true
        networks:
          - rede
        depends_on:
          - tomcat9
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
      tomcat9:
        image: tomcat-9
        hostname: tomcat9
        restart: unless-stopped
        ports:
          - "8080:8080"
        volumes:
          - type: bind
            source: ./catalina.sh
            target: /opt/tomcat/bin/catalina.sh
            read_only: true
          - type: bind
            source: ${WAR_POLARE}/polare.war # WAR_POLARE é o caminho da pasta "wars"
            target: /polare/deploy/polare.war
            read_only: true
          - type: bind
            source: ${WAR_POLARE}/polare-publico.war # WAR_POLARE é o caminho da pasta "wars"
            target: /polare/deploy/polare-publico.war
            read_only: true
        networks:
          - rede
    networks:
      rede:
        driver: bridge


.. warning::

    É necessário fazer a configuração SSL no nginx para distribuir a aplicação via HTTPS.


**nginx.conf** Configuração do nginx. É necessário fazer ajustes nesse arquivo em função das configurações do
ambiente local (as linhas com o comentário ``# ALTERAR`` devem ser modificadas).

.. code-block:: nginx

    proxy_cache_path /tmp/NGINX_treinamento_cache/ keys_zone=backcache:10m; # ALTERAR

    upstream polare {
        ip_hash;
        server tomcat9:8080;
    }

    server {
        listen 80;
        server_name polare-treinamento.ifpa.edu.br; # ALTERAR

        client_max_body_size 128M;

        access_log /var/log/nginx/polare-treinamento.ifpa.edu.br-80-access.log; # ALTERAR
        error_log /var/log/nginx/polare-treinamento.ifpa.edu.br-80-error.log; # ALTERAR

        # Redirect all HTTP to HTTPS
        location / {
            return 301 https://$server_name$request_uri;
        }
    }

    server {
        listen 443 ssl http2;
        server_name polare-treinamento.ifpa.edu.br; # ALTERAR

        client_max_body_size 128M;

        access_log /var/log/nginx/polare-treinamento.ifpa.edu.br-443-access.log; # ALTERAR
        error_log /var/log/nginx/polare-treinamento.ifpa.edu.br-443-error.log; # ALTERAR

        # certificados da instituição
        ssl_certificate /etc/nginx/ssl/nginx.crt; # ALTERAR
        ssl_certificate_key /etc/nginx/ssl/nginx.key; # ALTERAR

        ssl_session_cache   shared:SSL:1m;
        ssl_prefer_server_ciphers  on;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

        location = / {
            return 302 /polare;
        }

        location = /polare-publico/ {
            return 302 /polare-publico/relatorios;
        }

        location /polare/ {
            proxy_pass http://polare;
            proxy_cache backcache;
            proxy_set_header X-Real-IP  $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Real-Port $server_port;
            proxy_set_header X-Real-Scheme $scheme;
        }

        location /polare-publico/ {
            proxy_pass http://polare;
            proxy_cache backcache;
            proxy_set_header X-Real-IP  $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Real-Port $server_port;
            proxy_set_header X-Real-Scheme $scheme;
      }
    }

.. note::

    Para mais detalhes sobre a configuração no nginx acesse
    `https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04
    <https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04>`_


**catalina.sh** Arquivo fundamental referente aos paramêtros de configuração do sistema Polare. É necessário
fazer ajustes nesse arquivo em função das configurações do ambiente local (entre as linhas 334 e 380).

.. code-block:: bash
    :linenos:

    #!/bin/sh

    # Licensed to the Apache Software Foundation (ASF) under one or more
    # contributor license agreements.  See the NOTICE file distributed with
    # this work for additional information regarding copyright ownership.
    # The ASF licenses this file to You under the Apache License, Version 2.0
    # (the "License"); you may not use this file except in compliance with
    # the License.  You may obtain a copy of the License at
    #
    #     http://www.apache.org/licenses/LICENSE-2.0
    #
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an "AS IS" BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    # See the License for the specific language governing permissions and
    # limitations under the License.

    # -----------------------------------------------------------------------------
    # Control Script for the CATALINA Server
    #
    # For supported commands call "catalina.sh help" or see the usage section at
    # the end of this file.
    #
    # Environment Variable Prerequisites
    #
    #   Do not set the variables in this script. Instead put them into a script
    #   setenv.sh in CATALINA_BASE/bin to keep your customizations separate.
    #
    #   CATALINA_HOME   May point at your Catalina "build" directory.
    #
    #   CATALINA_BASE   (Optional) Base directory for resolving dynamic portions
    #                   of a Catalina installation.  If not present, resolves to
    #                   the same directory that CATALINA_HOME points to.
    #
    #   CATALINA_OUT    (Optional) Full path to a file where stdout and stderr
    #                   will be redirected.
    #                   Default is $CATALINA_BASE/logs/catalina.out
    #
    #   CATALINA_OUT_CMD (Optional) Command which will be executed and receive
    #                   as its stdin the stdout and stderr from the Tomcat java
    #                   process. If CATALINA_OUT_CMD is set, the value of
    #                   CATALINA_OUT will be used as a named pipe.
    #                   No default.
    #                   Example (all one line)
    #                   CATALINA_OUT_CMD="/usr/bin/rotatelogs -f $CATALINA_BASE/logs/catalina.out.%Y-%m-%d.log 86400"
    #
    #   CATALINA_OPTS   (Optional) Java runtime options used when the "start",
    #                   "run" or "debug" command is executed.
    #                   Include here and not in JAVA_OPTS all options, that should
    #                   only be used by Tomcat itself, not by the stop process,
    #                   the version command etc.
    #                   Examples are heap size, GC logging, JMX ports etc.
    #
    #   CATALINA_TMPDIR (Optional) Directory path location of temporary directory
    #                   the JVM should use (java.io.tmpdir).  Defaults to
    #                   $CATALINA_BASE/temp.
    #
    #   JAVA_HOME       Must point at your Java Development Kit installation.
    #                   Required to run the with the "debug" argument.
    #
    #   JRE_HOME        Must point at your Java Runtime installation.
    #                   Defaults to JAVA_HOME if empty. If JRE_HOME and JAVA_HOME
    #                   are both set, JRE_HOME is used.
    #
    #   JAVA_OPTS       (Optional) Java runtime options used when any command
    #                   is executed.
    #                   Include here and not in CATALINA_OPTS all options, that
    #                   should be used by Tomcat and also by the stop process,
    #                   the version command etc.
    #                   Most options should go into CATALINA_OPTS.
    #
    #   JAVA_ENDORSED_DIRS (Optional) Lists of of colon separated directories
    #                   containing some jars in order to allow replacement of APIs
    #                   created outside of the JCP (i.e. DOM and SAX from W3C).
    #                   It can also be used to update the XML parser implementation.
    #                   This is only supported for Java <= 8.
    #                   Defaults to $CATALINA_HOME/endorsed.
    #
    #   JPDA_TRANSPORT  (Optional) JPDA transport used when the "jpda start"
    #                   command is executed. The default is "dt_socket".
    #
    #   JPDA_ADDRESS    (Optional) Java runtime options used when the "jpda start"
    #                   command is executed. The default is localhost:8000.
    #
    #   JPDA_SUSPEND    (Optional) Java runtime options used when the "jpda start"
    #                   command is executed. Specifies whether JVM should suspend
    #                   execution immediately after startup. Default is "n".
    #
    #   JPDA_OPTS       (Optional) Java runtime options used when the "jpda start"
    #                   command is executed. If used, JPDA_TRANSPORT, JPDA_ADDRESS,
    #                   and JPDA_SUSPEND are ignored. Thus, all required jpda
    #                   options MUST be specified. The default is:
    #
    #                   -agentlib:jdwp=transport=$JPDA_TRANSPORT,
    #                       address=$JPDA_ADDRESS,server=y,suspend=$JPDA_SUSPEND
    #
    #   JSSE_OPTS       (Optional) Java runtime options used to control the TLS
    #                   implementation when JSSE is used. Default is:
    #                   "-Djdk.tls.ephemeralDHKeySize=2048"
    #
    #   CATALINA_PID    (Optional) Path of the file which should contains the pid
    #                   of the catalina startup java process, when start (fork) is
    #                   used
    #
    #   CATALINA_LOGGING_CONFIG (Optional) Override Tomcat's logging config file
    #                   Example (all one line)
    #                   CATALINA_LOGGING_CONFIG="-Djava.util.logging.config.file=$CATALINA_BASE/conf/logging.properties"
    #
    #   LOGGING_CONFIG  Deprecated
    #                   Use CATALINA_LOGGING_CONFIG
    #                   This is only used if CATALINA_LOGGING_CONFIG is not set
    #                   and LOGGING_CONFIG starts with "-D..."
    #
    #   LOGGING_MANAGER (Optional) Override Tomcat's logging manager
    #                   Example (all one line)
    #                   LOGGING_MANAGER="-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager"
    #
    #   UMASK           (Optional) Override Tomcat's default UMASK of 0027
    #
    #   USE_NOHUP       (Optional) If set to the string true the start command will
    #                   use nohup so that the Tomcat process will ignore any hangup
    #                   signals. Default is "false" unless running on HP-UX in which
    #                   case the default is "true"
    # -----------------------------------------------------------------------------

    # OS specific support.  $var _must_ be set to either true or false.
    cygwin=false
    darwin=false
    os400=false
    hpux=false
    case "`uname`" in
    CYGWIN*) cygwin=true;;
    Darwin*) darwin=true;;
    OS400*) os400=true;;
    HP-UX*) hpux=true;;
    esac

    # resolve links - $0 may be a softlink
    PRG="$0"

    while [ -h "$PRG" ]; do
      ls=`ls -ld "$PRG"`
      link=`expr "$ls" : '.*-> \(.*\)$'`
      if expr "$link" : '/.*' > /dev/null; then
        PRG="$link"
      else
        PRG=`dirname "$PRG"`/"$link"
      fi
    done

    # Get standard environment variables
    PRGDIR=`dirname "$PRG"`

    # Only set CATALINA_HOME if not already set
    [ -z "$CATALINA_HOME" ] && CATALINA_HOME=`cd "$PRGDIR/.." >/dev/null; pwd`

    # Copy CATALINA_BASE from CATALINA_HOME if not already set
    [ -z "$CATALINA_BASE" ] && CATALINA_BASE="$CATALINA_HOME"

    # Ensure that any user defined CLASSPATH variables are not used on startup,
    # but allow them to be specified in setenv.sh, in rare case when it is needed.
    CLASSPATH=

    if [ -r "$CATALINA_BASE/bin/setenv.sh" ]; then
      . "$CATALINA_BASE/bin/setenv.sh"
    elif [ -r "$CATALINA_HOME/bin/setenv.sh" ]; then
      . "$CATALINA_HOME/bin/setenv.sh"
    fi

    # For Cygwin, ensure paths are in UNIX format before anything is touched
    if $cygwin; then
      [ -n "$JAVA_HOME" ] && JAVA_HOME=`cygpath --unix "$JAVA_HOME"`
      [ -n "$JRE_HOME" ] && JRE_HOME=`cygpath --unix "$JRE_HOME"`
      [ -n "$CATALINA_HOME" ] && CATALINA_HOME=`cygpath --unix "$CATALINA_HOME"`
      [ -n "$CATALINA_BASE" ] && CATALINA_BASE=`cygpath --unix "$CATALINA_BASE"`
      [ -n "$CLASSPATH" ] && CLASSPATH=`cygpath --path --unix "$CLASSPATH"`
    fi

    # Ensure that neither CATALINA_HOME nor CATALINA_BASE contains a colon
    # as this is used as the separator in the classpath and Java provides no
    # mechanism for escaping if the same character appears in the path.
    case $CATALINA_HOME in
      *:*) echo "Using CATALINA_HOME:   $CATALINA_HOME";
          echo "Unable to start as CATALINA_HOME contains a colon (:) character";
          exit 1;
    esac
    case $CATALINA_BASE in
      *:*) echo "Using CATALINA_BASE:   $CATALINA_BASE";
          echo "Unable to start as CATALINA_BASE contains a colon (:) character";
          exit 1;
    esac

    # For OS400
    if $os400; then
      # Set job priority to standard for interactive (interactive - 6) by using
      # the interactive priority - 6, the helper threads that respond to requests
      # will be running at the same priority as interactive jobs.
      COMMAND='chgjob job('$JOBNAME') runpty(6)'
      system $COMMAND

      # Enable multi threading
      export QIBM_MULTI_THREADED=Y
    fi

    # Get standard Java environment variables
    if $os400; then
      # -r will Only work on the os400 if the files are:
      # 1. owned by the user
      # 2. owned by the PRIMARY group of the user
      # this will not work if the user belongs in secondary groups
      . "$CATALINA_HOME"/bin/setclasspath.sh
    else
      if [ -r "$CATALINA_HOME"/bin/setclasspath.sh ]; then
        . "$CATALINA_HOME"/bin/setclasspath.sh
      else
        echo "Cannot find $CATALINA_HOME/bin/setclasspath.sh"
        echo "This file is needed to run this program"
        exit 1
      fi
    fi

    # Add on extra jar files to CLASSPATH
    if [ ! -z "$CLASSPATH" ] ; then
      CLASSPATH="$CLASSPATH":
    fi
    CLASSPATH="$CLASSPATH""$CATALINA_HOME"/bin/bootstrap.jar

    if [ -z "$CATALINA_OUT" ] ; then
      CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out
    fi

    if [ -z "$CATALINA_TMPDIR" ] ; then
      # Define the java.io.tmpdir to use for Catalina
      CATALINA_TMPDIR="$CATALINA_BASE"/temp
    fi

    # Add tomcat-juli.jar to classpath
    # tomcat-juli.jar can be over-ridden per instance
    if [ -r "$CATALINA_BASE/bin/tomcat-juli.jar" ] ; then
      CLASSPATH=$CLASSPATH:$CATALINA_BASE/bin/tomcat-juli.jar
    else
      CLASSPATH=$CLASSPATH:$CATALINA_HOME/bin/tomcat-juli.jar
    fi

    # Bugzilla 37848: When no TTY is available, don't output to console
    have_tty=0
    if [ -t 0 ]; then
        have_tty=1
    fi

    # For Cygwin, switch paths to Windows format before running java
    if $cygwin; then
      JAVA_HOME=`cygpath --absolute --windows "$JAVA_HOME"`
      JRE_HOME=`cygpath --absolute --windows "$JRE_HOME"`
      CATALINA_HOME=`cygpath --absolute --windows "$CATALINA_HOME"`
      CATALINA_BASE=`cygpath --absolute --windows "$CATALINA_BASE"`
      CATALINA_TMPDIR=`cygpath --absolute --windows "$CATALINA_TMPDIR"`
      CLASSPATH=`cygpath --path --windows "$CLASSPATH"`
      [ -n "$JAVA_ENDORSED_DIRS" ] && JAVA_ENDORSED_DIRS=`cygpath --path --windows "$JAVA_ENDORSED_DIRS"`
    fi

    if [ -z "$JSSE_OPTS" ] ; then
      JSSE_OPTS="-Djdk.tls.ephemeralDHKeySize=2048"
    fi
    JAVA_OPTS="$JAVA_OPTS $JSSE_OPTS"

    # Register custom URL handlers
    # Do this here so custom URL handles (specifically 'war:...') can be used in the security policy
    JAVA_OPTS="$JAVA_OPTS -Djava.protocol.handler.pkgs=org.apache.catalina.webresources"

    # Check for the deprecated LOGGING_CONFIG
    # Only use it if CATALINA_LOGGING_CONFIG is not set and LOGGING_CONFIG starts with "-D..."
    if [ -z "$CATALINA_LOGGING_CONFIG" ]; then
      case $LOGGING_CONFIG in
        -D*) CATALINA_LOGGING_CONFIG="$LOGGING_CONFIG"
      esac
    fi

    # Set juli LogManager config file if it is present and an override has not been issued
    if [ -z "$CATALINA_LOGGING_CONFIG" ]; then
      if [ -r "$CATALINA_BASE"/conf/logging.properties ]; then
        CATALINA_LOGGING_CONFIG="-Djava.util.logging.config.file=$CATALINA_BASE/conf/logging.properties"
      else
        # Bugzilla 45585
        CATALINA_LOGGING_CONFIG="-Dnop"
      fi
    fi

    if [ -z "$LOGGING_MANAGER" ]; then
      LOGGING_MANAGER="-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager"
    fi

    # Set UMASK unless it has been overridden
    if [ -z "$UMASK" ]; then
        UMASK="0027"
    fi
    umask $UMASK

    # Java 9 no longer supports the java.endorsed.dirs
    # system property. Only try to use it if
    # JAVA_ENDORSED_DIRS was explicitly set
    # or CATALINA_HOME/endorsed exists.
    ENDORSED_PROP=ignore.endorsed.dirs
    if [ -n "$JAVA_ENDORSED_DIRS" ]; then
        ENDORSED_PROP=java.endorsed.dirs
    fi
    if [ -d "$CATALINA_HOME/endorsed" ]; then
        ENDORSED_PROP=java.endorsed.dirs
    fi

    # Make the umask available when using the org.apache.catalina.security.SecurityListener
    JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"

    if [ -z "$USE_NOHUP" ]; then
        if $hpux; then
            USE_NOHUP="true"
        else
            USE_NOHUP="false"
        fi
    fi
    unset _NOHUP
    if [ "$USE_NOHUP" = "true" ]; then
        _NOHUP="nohup"
    fi

    # Add the JAVA 9 specific start-up parameters required by Tomcat
    JDK_JAVA_OPTIONS="$JDK_JAVA_OPTIONS --add-opens=java.base/java.lang=ALL-UNNAMED"
    JDK_JAVA_OPTIONS="$JDK_JAVA_OPTIONS --add-opens=java.base/java.io=ALL-UNNAMED"
    JDK_JAVA_OPTIONS="$JDK_JAVA_OPTIONS --add-opens=java.base/java.util=ALL-UNNAMED"
    JDK_JAVA_OPTIONS="$JDK_JAVA_OPTIONS --add-opens=java.base/java.util.concurrent=ALL-UNNAMED"
    JDK_JAVA_OPTIONS="$JDK_JAVA_OPTIONS --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED"
    export JDK_JAVA_OPTIONS

    # configuração BANCO DE DADOS
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.datasource.username=postgres"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.datasource.password=postgres"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.datasource.url=jdbc:postgresql://polare-db:5432/polaredb"

    # configuração OAUTH  
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn-api.provider=ufrn"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn-api.client-id=ALTERAR" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn-api.client-secret=ALTERAR" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn-api.authorization-grant-type=client_credentials"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn.client-id=ALTERAR" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn.client-secret=ALTERAR" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn.scope=read"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn.authorization-grant-type=authorization_code"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.ufrn.redirect-uri=http://ALTERAR/polare/login/oauth2/code/ufrn" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.ufrn.authorization-uri=https://ALTERAR/authz-server/oauth/authorize" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.ufrn.token-uri=https://ALTERAR/authz-server/oauth/token" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.ufrn.user-info-uri=https://ALTERAR/security/v2/usuarios/me" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.ufrn.user-name-attribute=pessoa"
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.auth.logout.ufrn.logout-uri=https://ALTERAR/authz-server/j_spring_cas_security_logout?service=http://ALTERAR/polare-publico" # ALTERAR
    
    # configuração API serviços
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.api-key=ALTERAR"  # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.arquivos=https://ALTERAR/file/v1/arquivos" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.unidades=https://ALTERAR/unidade/v1/unidades" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.responsaveis=https://ALTERAR/pessoa/v1/responsaveis" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.servidor-localizacoes=https://ALTERAR/pessoa/v1/localizacoes-servidores" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.usuarios-sig=https://ALTERAR/usuario/v1/usuarios" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.servidores=https://ALTERAR/pessoa/v1/servidores" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.unidades-lotacao=https://ALTERAR/pessoa/v1/unidades-lotacao" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.unidades-exercicios=https://ALTERAR/pessoa/v1/unidades-exercicios" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.api.ufrn.services.unidades-localizacao=https://ALTERAR/pessoa/v1/unidades-localizacao" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.polare-url=http://ALTERAR/polare/login" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.polare-publico-url=http://ALTERAR/polare-publico" # ALTERAR

    # configuração GOVBR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.govbr.client-id=ALTERAR" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.govbr.client-secret=ALTERAR" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.govbr.scope=openid+\(email/phone\)+profile+govbr_empresa+govbr_confiabilidades"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.govbr.authorization-grant-type=authorization_code"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.registration.govbr.redirect-uri=http://ALTERAR/polare/login/oauth2/code/govbr" # ALTERAR
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.govbr.authorization-uri=https://sso.staging.acesso.gov.br/authorize"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.govbr.token-uri=https://sso.staging.acesso.gov.br/token"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.govbr.jwk-set-uri=https://sso.staging.acesso.gov.br/jwk"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.govbr.user-info-uri=https://sso.staging.acesso.gov.br/userinfo"
    CATALINA_OPTS="$CATALINA_OPTS -Dspring.security.oauth2.client.provider.govbr.user-name-attribute=name"
    CATALINA_OPTS="$CATALINA_OPTS -Dapp.auth.logout.govbr.logout-uri=https://sso.staging.acesso.gov.br/logout?post_logout_redirect_uri=http://ALTERAR/polare" # ALTERAR
    export CATALINA_OPTS

    # ----- Execute The Requested Command -----------------------------------------

    # Bugzilla 37848: only output this if we have a TTY
    if [ $have_tty -eq 1 ]; then
      echo "Using CATALINA_BASE:   $CATALINA_BASE"
      echo "Using CATALINA_HOME:   $CATALINA_HOME"
      echo "Using CATALINA_TMPDIR: $CATALINA_TMPDIR"
      if [ "$1" = "debug" ] ; then
        echo "Using JAVA_HOME:       $JAVA_HOME"
      else
        echo "Using JRE_HOME:        $JRE_HOME"
      fi
      echo "Using CLASSPATH:       $CLASSPATH"
      echo "Using CATALINA_OPTS:   $CATALINA_OPTS"
      if [ ! -z "$CATALINA_PID" ]; then
        echo "Using CATALINA_PID:    $CATALINA_PID"
      fi
    fi

    if [ "$1" = "jpda" ] ; then
      if [ -z "$JPDA_TRANSPORT" ]; then
        JPDA_TRANSPORT="dt_socket"
      fi
      if [ -z "$JPDA_ADDRESS" ]; then
        JPDA_ADDRESS="localhost:8000"
      fi
      if [ -z "$JPDA_SUSPEND" ]; then
        JPDA_SUSPEND="n"
      fi
      if [ -z "$JPDA_OPTS" ]; then
        JPDA_OPTS="-agentlib:jdwp=transport=$JPDA_TRANSPORT,address=$JPDA_ADDRESS,server=y,suspend=$JPDA_SUSPEND"
      fi
      CATALINA_OPTS="$JPDA_OPTS $CATALINA_OPTS"
      shift
    fi

    if [ "$1" = "debug" ] ; then
      if $os400; then
        echo "Debug command not available on OS400"
        exit 1
      else
        shift
        if [ "$1" = "-security" ] ; then
          if [ $have_tty -eq 1 ]; then
            echo "Using Security Manager"
          fi
          shift
          eval exec "\"$_RUNJDB\"" "\"$CATALINA_LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
            -D$ENDORSED_PROP="$JAVA_ENDORSED_DIRS" \
            -classpath "$CLASSPATH" \
            -sourcepath "$CATALINA_HOME"/../../java \
            -Djava.security.manager \
            -Djava.security.policy=="$CATALINA_BASE"/conf/catalina.policy \
            -Dcatalina.base="$CATALINA_BASE" \
            -Dcatalina.home="$CATALINA_HOME" \
            -Djava.io.tmpdir="$CATALINA_TMPDIR" \
            org.apache.catalina.startup.Bootstrap "$@" start
        else
          eval exec "\"$_RUNJDB\"" "\"$CATALINA_LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
            -D$ENDORSED_PROP="$JAVA_ENDORSED_DIRS" \
            -classpath "$CLASSPATH" \
            -sourcepath "$CATALINA_HOME"/../../java \
            -Dcatalina.base="$CATALINA_BASE" \
            -Dcatalina.home="$CATALINA_HOME" \
            -Djava.io.tmpdir="$CATALINA_TMPDIR" \
            org.apache.catalina.startup.Bootstrap "$@" start
        fi
      fi

    elif [ "$1" = "run" ]; then

      shift
      if [ "$1" = "-security" ] ; then
        if [ $have_tty -eq 1 ]; then
          echo "Using Security Manager"
        fi
        shift
        eval exec "\"$_RUNJAVA\"" "\"$CATALINA_LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
          -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
          -classpath "\"$CLASSPATH\"" \
          -Djava.security.manager \
          -Djava.security.policy=="\"$CATALINA_BASE/conf/catalina.policy\"" \
          -Dcatalina.base="\"$CATALINA_BASE\"" \
          -Dcatalina.home="\"$CATALINA_HOME\"" \
          -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
          org.apache.catalina.startup.Bootstrap "$@" start
      else
        eval exec "\"$_RUNJAVA\"" "\"$CATALINA_LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
          -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
          -classpath "\"$CLASSPATH\"" \
          -Dcatalina.base="\"$CATALINA_BASE\"" \
          -Dcatalina.home="\"$CATALINA_HOME\"" \
          -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
          org.apache.catalina.startup.Bootstrap "$@" start
      fi

    elif [ "$1" = "start" ] ; then

      if [ ! -z "$CATALINA_PID" ]; then
        if [ -f "$CATALINA_PID" ]; then
          if [ -s "$CATALINA_PID" ]; then
            echo "Existing PID file found during start."
            if [ -r "$CATALINA_PID" ]; then
              PID=`cat "$CATALINA_PID"`
              ps -p $PID >/dev/null 2>&1
              if [ $? -eq 0 ] ; then
                echo "Tomcat appears to still be running with PID $PID. Start aborted."
                echo "If the following process is not a Tomcat process, remove the PID file and try again:"
                ps -f -p $PID
                exit 1
              else
                echo "Removing/clearing stale PID file."
                rm -f "$CATALINA_PID" >/dev/null 2>&1
                if [ $? != 0 ]; then
                  if [ -w "$CATALINA_PID" ]; then
                    cat /dev/null > "$CATALINA_PID"
                  else
                    echo "Unable to remove or clear stale PID file. Start aborted."
                    exit 1
                  fi
                fi
              fi
            else
              echo "Unable to read PID file. Start aborted."
              exit 1
            fi
          else
            rm -f "$CATALINA_PID" >/dev/null 2>&1
            if [ $? != 0 ]; then
              if [ ! -w "$CATALINA_PID" ]; then
                echo "Unable to remove or write to empty PID file. Start aborted."
                exit 1
              fi
            fi
          fi
        fi
      fi

      shift
      if [ -z "$CATALINA_OUT_CMD" ] ; then
        touch "$CATALINA_OUT"
      else
        if [ ! -e "$CATALINA_OUT" ]; then
          if ! mkfifo "$CATALINA_OUT"; then
            echo "cannot create named pipe $CATALINA_OUT. Start aborted."
            exit 1
          fi
        elif [ ! -p "$CATALINA_OUT" ]; then
          echo "$CATALINA_OUT exists and is not a named pipe. Start aborted."
          exit 1
        fi
        $CATALINA_OUT_CMD <"$CATALINA_OUT" &
      fi
      if [ "$1" = "-security" ] ; then
        if [ $have_tty -eq 1 ]; then
          echo "Using Security Manager"
        fi
        shift
        eval $_NOHUP "\"$_RUNJAVA\"" "\"$CATALINA_LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
          -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
          -classpath "\"$CLASSPATH\"" \
          -Djava.security.manager \
          -Djava.security.policy=="\"$CATALINA_BASE/conf/catalina.policy\"" \
          -Dcatalina.base="\"$CATALINA_BASE\"" \
          -Dcatalina.home="\"$CATALINA_HOME\"" \
          -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
          org.apache.catalina.startup.Bootstrap "$@" start \
          >> "$CATALINA_OUT" 2>&1 "&"

      else
        eval $_NOHUP "\"$_RUNJAVA\"" "\"$CATALINA_LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
          -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
          -classpath "\"$CLASSPATH\"" \
          -Dcatalina.base="\"$CATALINA_BASE\"" \
          -Dcatalina.home="\"$CATALINA_HOME\"" \
          -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
          org.apache.catalina.startup.Bootstrap "$@" start \
          >> "$CATALINA_OUT" 2>&1 "&"

      fi

      if [ ! -z "$CATALINA_PID" ]; then
        echo $! > "$CATALINA_PID"
      fi

      echo "Tomcat started."

    elif [ "$1" = "stop" ] ; then

      shift

      SLEEP=5
      if [ ! -z "$1" ]; then
        echo $1 | grep "[^0-9]" >/dev/null 2>&1
        if [ $? -gt 0 ]; then
          SLEEP=$1
          shift
        fi
      fi

      FORCE=0
      if [ "$1" = "-force" ]; then
        shift
        FORCE=1
      fi

      if [ ! -z "$CATALINA_PID" ]; then
        if [ -f "$CATALINA_PID" ]; then
          if [ -s "$CATALINA_PID" ]; then
            kill -0 `cat "$CATALINA_PID"` >/dev/null 2>&1
            if [ $? -gt 0 ]; then
              echo "PID file found but either no matching process was found or the current user does not have permission to stop the process. Stop aborted."
              exit 1
            fi
          else
            echo "PID file is empty and has been ignored."
          fi
        else
          echo "\$CATALINA_PID was set but the specified file does not exist. Is Tomcat running? Stop aborted."
          exit 1
        fi
      fi

      eval "\"$_RUNJAVA\"" $LOGGING_MANAGER "$JAVA_OPTS" \
        -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
        -classpath "\"$CLASSPATH\"" \
        -Dcatalina.base="\"$CATALINA_BASE\"" \
        -Dcatalina.home="\"$CATALINA_HOME\"" \
        -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
        org.apache.catalina.startup.Bootstrap "$@" stop

      # stop failed. Shutdown port disabled? Try a normal kill.
      if [ $? != 0 ]; then
        if [ ! -z "$CATALINA_PID" ]; then
          echo "The stop command failed. Attempting to signal the process to stop through OS signal."
          kill -15 `cat "$CATALINA_PID"` >/dev/null 2>&1
        fi
      fi

      if [ ! -z "$CATALINA_PID" ]; then
        if [ -f "$CATALINA_PID" ]; then
          while [ $SLEEP -ge 0 ]; do
            kill -0 `cat "$CATALINA_PID"` >/dev/null 2>&1
            if [ $? -gt 0 ]; then
              rm -f "$CATALINA_PID" >/dev/null 2>&1
              if [ $? != 0 ]; then
                if [ -w "$CATALINA_PID" ]; then
                  cat /dev/null > "$CATALINA_PID"
                  # If Tomcat has stopped don't try and force a stop with an empty PID file
                  FORCE=0
                else
                  echo "The PID file could not be removed or cleared."
                fi
              fi
              echo "Tomcat stopped."
              break
            fi
            if [ $SLEEP -gt 0 ]; then
              sleep 1
            fi
            if [ $SLEEP -eq 0 ]; then
              echo "Tomcat did not stop in time."
              if [ $FORCE -eq 0 ]; then
                echo "PID file was not removed."
              fi
              echo "To aid diagnostics a thread dump has been written to standard out."
              kill -3 `cat "$CATALINA_PID"`
            fi
            SLEEP=`expr $SLEEP - 1 `
          done
        fi
      fi

      KILL_SLEEP_INTERVAL=5
      if [ $FORCE -eq 1 ]; then
        if [ -z "$CATALINA_PID" ]; then
          echo "Kill failed: \$CATALINA_PID not set"
        else
          if [ -f "$CATALINA_PID" ]; then
            PID=`cat "$CATALINA_PID"`
            echo "Killing Tomcat with the PID: $PID"
            kill -9 $PID
            while [ $KILL_SLEEP_INTERVAL -ge 0 ]; do
                kill -0 `cat "$CATALINA_PID"` >/dev/null 2>&1
                if [ $? -gt 0 ]; then
                    rm -f "$CATALINA_PID" >/dev/null 2>&1
                    if [ $? != 0 ]; then
                        if [ -w "$CATALINA_PID" ]; then
                            cat /dev/null > "$CATALINA_PID"
                        else
                            echo "The PID file could not be removed."
                        fi
                    fi
                    echo "The Tomcat process has been killed."
                    break
                fi
                if [ $KILL_SLEEP_INTERVAL -gt 0 ]; then
                    sleep 1
                fi
                KILL_SLEEP_INTERVAL=`expr $KILL_SLEEP_INTERVAL - 1 `
            done
            if [ $KILL_SLEEP_INTERVAL -lt 0 ]; then
                echo "Tomcat has not been killed completely yet. The process might be waiting on some system call or might be UNINTERRUPTIBLE."
            fi
          fi
        fi
      fi

    elif [ "$1" = "configtest" ] ; then

        eval "\"$_RUNJAVA\"" $LOGGING_MANAGER "$JAVA_OPTS" \
          -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
          -classpath "\"$CLASSPATH\"" \
          -Dcatalina.base="\"$CATALINA_BASE\"" \
          -Dcatalina.home="\"$CATALINA_HOME\"" \
          -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
          org.apache.catalina.startup.Bootstrap configtest
        result=$?
        if [ $result -ne 0 ]; then
            echo "Configuration error detected!"
        fi
        exit $result

    elif [ "$1" = "version" ] ; then

        "$_RUNJAVA"   \
          -classpath "$CATALINA_HOME/lib/catalina.jar" \
          org.apache.catalina.util.ServerInfo

    else

      echo "Usage: catalina.sh ( commands ... )"
      echo "commands:"
      if $os400; then
        echo "  debug             Start Catalina in a debugger (not available on OS400)"
        echo "  debug -security   Debug Catalina with a security manager (not available on OS400)"
      else
        echo "  debug             Start Catalina in a debugger"
        echo "  debug -security   Debug Catalina with a security manager"
      fi
      echo "  jpda start        Start Catalina under JPDA debugger"
      echo "  run               Start Catalina in the current window"
      echo "  run -security     Start in the current window with security manager"
      echo "  start             Start Catalina in a separate window"
      echo "  start -security   Start in a separate window with security manager"
      echo "  stop              Stop Catalina, waiting up to 5 seconds for the process to end"
      echo "  stop n            Stop Catalina, waiting up to n seconds for the process to end"
      echo "  stop -force       Stop Catalina, wait up to 5 seconds and then use kill -KILL if still running"
      echo "  stop n -force     Stop Catalina, wait up to n seconds and then use kill -KILL if still running"
      echo "  configtest        Run a basic syntax check on server.xml - check exit code for result"
      echo "  version           What version of tomcat are you running?"
      echo "Note: Waiting for the process to end and use of the -force option require that \$CATALINA_PID is defined"
      exit 1

    fi


O seguinte comando deve ser executado no diretório que contém os arquivos anteriores para criar os contêiners
e infraestrutura necessária para o funcionamento do sistema Polare:

.. code-block:: bash

    docker-compose up -d


.. figure:: /_static/img/login-polare.png
    :align: center

    Tela de login do sistema Polare


.. figure:: /_static/img/publico-polare.png
    :align: center

    Área pública sistema Polare


.. warning::

    O login no sistema Polare só poderá ser efetuado se uma infraestrutura OAUTH estiver instalada e
    configurada. As duas opções atualmente são a infraestrutura fornecida pela UFRN (descrita na sessão
    Pré-Requisitos de Instalação), ou o login único pelo GOVBR.