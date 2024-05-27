Obtenção e configuração das credenciais para o login único GOVBR
================================================================

As credenciais de acesso para o login único via GOVBR podem ser obtidas enviando o documento referente ao
plano de integração da instituição para o ministério da economia. Em seguida as credenciais são utilizadas
na configuração dos parâmetros de funcionamento do sistema Polare.

.. warning::

    As credenciais para login único GOVBR são liberadas inicialmente para o ambiente de **homologação**.
    Somente após a implantação e funcionamento do ambiente de homologação é que as credenciais para o ambiente
    de **produção** poderão ser solicitadas.


.. warning::

    As credenciais de acesso, nessa versão da documentação, foram enviadas em arquivo criptografado por email.
    É necessário descriptografar o arquivo contendo as credenciais para acessá-las. A instrução de como
    efetuar esse procedimento foi informada no mesmo email que contém o arquivo com as credenciais.


Plano de Integração
-------------------

A documentação referente a obtenção das credenciais para o login único GOVBR pode acessada no *link* a seguir:

    `https://manual-roteiro-integracao-login-unico.servicos.gov.br/pt/stable/solicitarconfiguracao.html#solicitacao-de-configuracao <https://manual-roteiro-integracao-login-unico.servicos.gov.br/pt/stable/solicitarconfiguracao.html#solicitacao-de-configuracao>`_


Clique :download:`aqui <_downloads/integracao-polare-govbr-homologacao-template.docx>` para obter um modelo de
documento pré-preenchido para facilitar a obtenção das credenciais.

É necessário assinar o documento digitalmente antes de enviá-lo por email. O assinador utilizado foi o do GOVBR:

    `https://assinador.iti.br/assinatura/index.xhtml <https://assinador.iti.br/assinatura/index.xhtml>`_


.. warning::

    Alguns assinadores removem assinaturas que já foram gravadas em um documento. Nesses casos é recomendável
    assinar o documento primeiro utilizando esses assinadores e posteriormente utilizar o assinador do GOVBR,
    visto que o último não remove assinaturas gravadas no documento.


.. note::

    O modelo de documento citado anteriormente está no formato ``.doc``. Nesta implantação, o documento (Plano
    de Integração) enviado por email para o ministério da economia foi primeiramente convertido para ``.pdf``
    e depois assinado digitalmente pelos participantes.


Configuração
------------

As credenciais de acesso (*client_id* e *secret*) precisam ser definidas nos parâmetros de configuração do
sistema do Polare. Nesta documentação, os paramêtros de configuração estão definidos no arquivo
``catalina.sh``. Detalhe para as linhas 2 e 3 no trecho de código a seguir:

.. code-block:: bash
    :linenos:
    :emphasize-lines: 2,3,6,12

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


.. note::

    Os paramêtros de configuração do sistema Polare podem ser definidos de outras formas, como por exemplo,
    variáveis de ambiente. Nesta documentação escolhemos definir esses parâmetros no arquivo ``catalina.sh``
    por conveniência.
