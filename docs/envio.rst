API PGD
=======

O envio de dados (planos individuais) para o governo é feito por uma subaplicação externa ao sistema Polare
disponível no endereço `https://github.com/djangonauta/dj-polare-pgd
<https://github.com/djangonauta/dj-polare-pgd>`_. Essa aplicação também é utilizada para visualizar a
formatação dos dados a serem enviados e pode ser utilizada para consultas e análise de planos individuais.

Instalação
----------

Como se trata de um aplicação `django <https://www.djangoproject.com/>`_ / `python <https://www.python.org/>`_
disponibilizada no `github <https://github.com/>`_, os seguintes passos para a instalação são necessários:

    1. Instalação do interpretador python e as dependências do ambiente
    2. Instalação do Git e clonagem do projeto
    3. Instalação e configuração das dependências do Projeto


Instalação do interpretador python e as dependências do ambiente
****************************************************************

As versões mais novas de sistemas Linux (como `Ubuntu <https://ubuntu.com/>`_) provavelmente já possuem o
interpretador Python3 instalado. A aplicação citada anteriormente foi testada utilizando a versão mais recente
da linguagem Python no momento que esta documentação é escrita (Versão 3.11.2).

Para gerenciar múltiplas versões do interpretador Python (incluindo as versões mais novas) recomendamos
utilizar `Pyenv <https://github.com/pyenv/pyenv>`_.

O comando a seguir instala as dependências (pacotes Python) do ambiente utilizando o gerenciador de pacotes
`Pip <https://pypi.org/project/pip/>`_:

.. code-block:: bash
    :linenos:

    pip3 install -U pip setuptools wheel bpython


Instalação do Git e clonagem do projeto
***************************************

O Git pode ser encontrado no endereço `https://git-scm.com/downloads <https://git-scm.com/downloads>`_. Em
sistemas linux (Ubuntu) o seguinte comando pode ser utilizado para instalá-lo:

.. code-block:: bash
    :linenos:

    sudo apt install git -y


Para clonar o projeto o seguinte comando pode ser utilizado:

.. code-block:: bash
    :linenos:

    git clone git@github.com:djangonauta/dj-polare-pgd.git


.. note:: O comando anterior baixa e salva o projeto em um diretório chamado ``dj-polare-pgd``


Instalação e configuração das dependências do projeto
*****************************************************

As configurações do projeto são feitas via variáveis de ambiente e/ou adicionando as seguintes variáveis no
arquivo localizado em ``dj-polare-pgd/projeto/settings/.env``:

.. code-block:: bash

    SECRET_KEY='3yzj!k%^^^c*k5is(2^)wlvp-z65p)o5yq@^v+d$)ny^9pk1y_'
    DATABASE_URL='postgres://postgres:admin@localhost:5432/polaredb-backup'
    ADMINS='usuario=usuario@domain.edu.br'
    EMAIL_URL='consolemail://:@'
    API_PGD_LOGIN_URL='http://localhost:5057/auth/jwt/login'
    API_PGD_PLANO_TRABALHO_URL='http://localhost:5057/plano_trabalho'
    API_PGD_USERNAME='usuario@domain.edu.br'
    API_PGD_PASSWORD='senha'


As URLs referentes a configurações contendo usuário e senha possuem o seguinte formato:
``protocolo://usuario:senha@host:porta/recurso``.

A variável SECRET_KEY é necessária para o funcionamento do framework Django especialmente relacionada com
segurança. O seguinte site pode ser utilizado para gerar essa string: `https://djecrety.ir/
<https://djecrety.ir/>`_.

A variável DATABASE_URL contém as configurações de acesso ao banco de dados postgresql. Recomendamos que
inicialmente seja utilizada uma nova base (backup) do sistema Polare. A configuração apresentada tenta
conectar em um banco de dados postgresql chamado ``polaredb-backup`` rodando em ``localhost`` na porta
``5432`` utilizando o usuário ``postgres`` e a senha ``admin``. O responsável pela distribuição da aplicação
deve alterar esses valores conforme as especificidades de sua instituição.

A variável ADMINS configura os usuários admin do sistema no formato
``usuario1=usuario1@domain.edu.br,usuario2=usuario2@domain.edu.br``. Esses usuários normalmente são
notificados por email quando erros acontecem no sistema.

A variável EMAIL_URL configura o servidor de email utilizado para enviar mensagens. Essa aplicação no momento
não trabalha enviando emails. A configuração apresentada configura o sistema para enviar mensagens para a
linha de comando.

As variáveis API_PGD_*_URL são utilizadas para conexão e envio de dados para a API PGD do governo. As URLS
apresentadas apontam para o ambiente local de desenvolvimento dessa api descrito em
`https://github.com/economiagovbr/api-pgd <https://github.com/economiagovbr/api-pgd>`_. A variáveis
API_PGD_USERNAME e API_PGD_PASSWORD devem ser definidas conforme cadastradas no ambiente de desenvolvimento,
ou obtidas para o ambiente de produção. As credenciais de produção podem ser obtidas na documentação oficial
`https://api-programadegestao.economia.gov.br/docs <https://api-programadegestao.economia.gov.br/docs>`_

.. note::
    Para configurar e levantar localmente a aplicação de envio de dados para testes da API PGD acesse a
    documentação oficial disponível em `https://github.com/economiagovbr/api-pgd <https://github.com/economiagovbr/api-pgd>`_

As dependências do projeto são gerenciadas utilizando `pipenv <https://pipenv.pypa.io/en/latest/>`_.
Inicialmente deve ser criado o ambiente virtual a partir do diretório ``dj-polare-pgd`` utilizando o seguinte
comando:

.. code-block:: bash
    :linenos:

    pipenv shell


.. note::
    O comando anterior também ativa o ambiente virtual.


Após a criação do ambiente virtual as dependências podem ser instaladas com o comando:

.. code-block:: bash
    :linenos:

    pipenv install --dev


.. note::
    No ambiente de produção, como as dependências de desenvolvimento não são necessárias, o comando seria
    ``pipenv install``.


Execução da aplicação
---------------------

Com as dependências instaladas e as váriaveis de ambiente configuradas é possível executar a aplicação no modo
desenvolvimento com o comando:

.. code-block:: bash
    :linenos:

    invoke


.. note::
    O comando acima executa a task default pyinvoke chamada "run_server" (definida no arquivo tasks.py).
    Muitas tasks pyinvoke servem para executar outros comandos por por conveniência. É possível executar a
    aplicação utilizando o comando padrão django ``./manage.py runserver 0.0.0.0:8000 --settings projeto.settings.development``


A aplicação deve estar rodando em ``localhost:8000``:

.. figure:: /_static/img/envio/api-root.png
    :align: center


É necessário criar um usuário administrador para visualizar os endpoints contendo os planos individuais
serializados no formato json.

Para criar um usuário administrador deve-se utilizar o seguinte comando e fornecer os dados que o utilitário
necessita (usuário, email e senha):

.. code-block:: bash
    :linenos:

    ./manage.py createsuperuser


.. note::
    O comando acima pode ser utilizado no mesmo terminal que executou a aplicação, mas para tanto a mesma
    precisa ser finalizada com o atalho no teclado Control + C


.. warning::
    É necessário que o ambiente virtual esteja ativado e com as dependências instaladas para que os comandos
    relacionados com a aplicação funcionem corretamente.


Com pelo menos um usuário admin criado é possível logar na aplicação clicando no link login no canto superior
direito:

.. figure:: /_static/img/envio/login.png
    :align: center


Após logar na aplicação é possível visualizar os planos individuais no formato json esperado pela API PGD do
governo clicando no link `http://localhost:8000/api/v1/plano-individual/
<http://localhost:8000/api/v1/plano-individual/>`_:

.. figure:: /_static/img/envio/planos.png
    :align: center


É possível fazer consultas de planos individuais clicando no botão "filter". A aplicação busca pelas chaves
matricula_siape, nome_participante e modalidade_execucao (presencial, remoto e híbrido - valores 1, 2 e 3
respectivamente).

Envio de dados
--------------

O envio de dados para a API PGD do governo é feita utilizando o seguinte comando:

.. code-block:: bash
    :linenos:

    ./manage.py enviar_dados


.. note::
    O comando anterior consulta todos os planos individuais ativos e faz solicitações http assíncronas utilizando
    as urls definidas na seção de configuração (API_PGD_LOGIN_URL e API_PGD_PLANO_TRABALHO_URL).


No momento que esta documentação é escrita ocorre um erro ao enviar os dados para o banco de dados da API PGD
localmente (desenvolvimento). Esse erro acontece porque alguns código de unidade do instituto são muito
longos, ultrapassando o limite do tipo de dados inteiro da coluna cod_unidade_exercicio da tabela
public.plano_trabalho do banco api_pgd.

Para contornar esse problema localmente, o tipo de dados da referida coluna foi alterado de int para bigint
utilizando a SQL a seguir (conectado no banco da aplicação de recebimento de dados api_pgd):

.. code-block:: bash
    :linenos:

    api_pgd=# alter table public.plano_trabalho alter cod_unidade_exercicio type bigint;
