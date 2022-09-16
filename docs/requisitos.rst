.. _pre_requisitos:

Pré-Requisitos de Instalação
============================

É recomendável que a equipe técnica responsável pela implantação no sistema Polare na instituição tenha alguma
experiência com as tecnologias a seguir.

**Java.** O sistema Polare está sendo escrito em `Java <https://java.com>`_ e portanto alguma experiência com
essa linguagem pode ser útil, mas não estritamente necessário para implantaçao do sistema.

**Maven.** O sistema Polare utiliza uma ferramenta construção de projeto e gerenciamento de dependências
chamada `Maven <https://maven.apache.org/>`_. Não é necessário ter conhecimento aprofundado sobre essa
ferramenta, visto que para gerar os artefatos ``.war`` do sistema Polare é preciso apenas ter o Maven
instalado e executar comandos pré-definidos no *shell* do sistema.

**Git.** Git é um sistema de versionamento (controle de versão) distribuído. Necessário para obtenção do
código fonte do sistema Polare. Não é necessário ter conhecimento aprofundado sobre essa ferramenta, visto que
para obter o código fonte do sistema Polare é preciso apenas ter o Git instalado e executar comandos
pré-definidos no *shell* do sistema.

**Web.** Pode ser necessário editar algumas páginas *web* do sistema Polare de forma a customizar a identidade
visual da instituição. O sistema Polare utiliza o *framework web* `Thymeleaf <https://www.thymeleaf.org/>`_,
mas algum conhecimento básico de html, css e javascript deve ser suficiente para efetuar alterações em páginas
*web* do sistema.

**Linux.** Experiência fundamental. Nessa implantação utilizamos o `Ubuntu <https://ubuntu.com/>`_ linux por
conveniência. Outras *distros* linux podem ser utilizadas de acordo com a necessidade da instituição.

**Docker.** Nesta implantação optamos por utilizar `Docker <https://docs.docker.com/get-docker/>`_, embora
cada instituição possa definir como o sistema Polare deverá ser implantado internamente. Entretanto,
recomendamos Docker como uma forma moderna e robusta de executar esse tipo de implantação.

**Postgresql.** Experiência com bancos de dados é importante em virtualmente qualquer sistema moderno de
*sofware*. Não é estritamente necessária para a implantação do sistema Polare, mas é útil para entender e
resolver certos problemas relacionados com migrações e *schemas*, bem como configuração de usuários e acesso
ao banco de dados.

Software e Hardware necessários
-------------------------------

**Infraestrutura de API**. É necessário instalar uma infraestrutura de que permita acessar a API de sistemas.
Essa API de sistemas é responsável por fornecer ao sistema Polare informações do usuário servidor, como por
exemplo, lotação e hierarquia funcional. Tecnicamente, essa infraestrutura é um conjunto de sistemas a parte
que fornece informações armazenadas nos bancos de dados do SIG via API `Restful
<https://en.wikipedia.org/wiki/Representational_state_transfer>`_.

A UFRN pode disponibilizar um ambiente contendo essa infraestrutura. Acesse a documentação oficial (é
necessário login) para mais detalhes:

    https://docs.info.ufrn.br/doku.php?id=cooperacao:tutoriais:infraestrutura:api:montagem_infraestrutura_api.

.. figure:: /_static/img/visao-geral-ambiente-api.png
    :align: center

    Visão geral do ambiente


.. note::

    A infraestrutura de API utilizada pelo Polare no IFPA foi instalada anteriormente para ser utilizada pelo
    SIGAA Mobile. Nesse caso a infraestrutura de API foi reutilizada.


.. warning::

    Alguns erros de API no sistema Polare podem ser corrigidos atualizando as imagens Docker de uma
    infraestrutura de API previamente instalada. O problema decorre da defasagem entre a versão da
    infraestrutura de API instalada e o Polare.


**Maven.** Instalar o Maven. Basicamente fazer o download da ferramenta e acessar os executáveis contidos no
diretório bin via linha de comando. Para mais detalhes acesse `https://maven.apache.org/download.cgi <https://maven.apache.org/download.cgi>`_

**Git.** Instalar o Git. Para mais detalhes acesse `https://git-scm.com/download/linux <https://git-scm.com/download/linux>`_

**Servidor Linux.** Instalar um servidor Linux com seguintes configurações:

   - 4 GB de memória Ram
   - 100 GB de armazenamento
   - 4 núcleos de processador
   - Ubuntu 22.04.1 LTS


**Docker e Docker-compose**. Instalar no servidor Linux as versões mais atuais do `Docker
<https://docs.docker.com/get-docker/>`_ e `Docker-compose <https://docs.docker.com/compose/>`_.