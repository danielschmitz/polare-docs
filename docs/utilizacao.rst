O projeto
=========

A Coordenação-Geral de Desenvolvimento de Pessoas da Rede Federal (CGDP) vinculada à Diretoria de
Desenvolvimento da Rede Federal de Educação Profissional, Científica e Tecnológica (DDR) da Secretaria de
Educação Profissional e Tecnológica (SETEC) apoiou o Projeto de Pesquisa Aplicada para o Desenvolvimento de um
Sistema Informatizado para atender o Programa de Gestão e Desempenho (PGD) das instituições que utilização do
Sistema Integrado de Gestão de Pessoas – SIGRH vinculadas a SETEC/MEC.

Após levantamento das versões do sistema SIGRH em cada uma das instituições vinculadas a SETEC/MEC e
considerando os aspectos tecnológicos o projeto se apresentou viável para uma solução (Orbital) do SIGRH, ou
seja, utilizando a Base de Dados do SIGRH. Para a continuidade do projeto o mesmo foi alinhado ao projeto de
PGD da UFRN com a solução do sistema orbital Polare.

Neste alinhamento entre os projetos só estão previstos os módulos (Plano Gerencial e Plano Individual) que
atende a minuta disponibilizada pela CGDP/DDR/SETEC/MEC com as recomendações mínimas para um Sistemas
Informatizados para atender os Programas de Gestão conforme Instrução Normativa SGP/ME nº 65, de 30/07/2020 e
Nota Técnica Conjunta para Atos Normativos SEI nº 11/2020/ME.

O Sistema Polare (Versão 0.5)
=============================

Apresentação
------------

O sistema Polare foi desenvolvido para atender as necessidades das instituições em relação ao seu PGD. O
Sistema utiliza a base de dados do Sistema Integrado de Gestão de Pessoas – SIGRH. Como já é de conhecimento
das instituições que utilizam os sistemas da UFRN a sua utilização está baseada na Resolução nº
051/2020-CONSAD, que disciplina o relacionamento entre a UFRN e os interessados no licenciamento e
transferência dos Sistemas Integrados de Gestão - SIG-UFRN bem como sua manutenção, evolução e
sustentabilidade.

Hierarquia Organizacional
-------------------------
	
O Polare utiliza a estrutura organizacional da instituição (organograma), ou seja, representa a forma pela
qual as Atribuições da Unidade serão desenvolvidas em cada setor conforme gráficos hierárquicos. A hierarquia
institucional atualizada e bem estruturada é importante para refletir no sistema os vínculos ativos de cada
servidor. Nesta versão (0.5) do Polare, dois módulos foram desenvolvidos para atender as necessidades IN nº
65/2020, e foram denominados de Plano Gerencial e Plano Individual.
	
Fluxo Geral dos Módulos
-----------------------
	
O Polare inicialmente possui em seu fluxo geral 04 (quatro) perfis de usuários conforme figura 01.

.. figure:: /_static/img/utilizacao/01_Figura_Fluxo.png
    :align: center

    Fluxo Geral dos Módulos do Polare


Administrador Polare
--------------------

Perfil destinado a equipe de TI que permitirá realizar as configurações iniciais do sistema, como habilitar a
funcionalidade para os Gestores do PGD na instituição cadastrar quais unidades serão Auto-homologáveis, ou
seja, não necessita de um Dirigente de Unidade para homologar seus planos.


Dirigente da Unidade
--------------------

Perfil destinado ao Gestor responsável pela condução de unidades de administração executiva, acadêmica e
suplementar. Compete ao dirigente a elaboração do Plano Estratégico da Unidade de Administração, supervisionar
os acompanhamentos das atividades de sua unidade, acompanhar os resultados obtidos e elaborar relatório anual
de avaliação da unidade. Nota 01: Nesta versão do sistema não foi previsto Módulo para Plano Estratégico.


Cheﬁa da Unidade
----------------

Perfil destinado a autoridade imediatamente superior ao servidor. Compete à Cheﬁa imediata a elaboração do
Plano Gerencial da Unidade de Localização, acompanhar a adaptação e o desempenho dos servidores no Programa de
Gestão e Desempenho (PGD), homologar o Plano Individual do servidor e veriﬁcar o cumprimento das entregas.


Servidor
--------

Perfil que permite a elaboração do Plano Individual do Servidor, incluindo cadastro de entregas. Nota 02:
Nesta versão do sistema só foi habilitado o perfil para servidores TAE, o perfil para servidores docentes com
função de chefia ainda será desenvolvido.


Unidade Autohomologável
=======================

Funcionalidade que permite incluir unidades da instituição como unidade auto-homologável, ao qual essa unidade
após o cadastro do   Plano Gerencial da Unidade de Localização, não precisa passar por uma homologação do
dirigente da unidade imediatamente superior, o que signiﬁca que todos os Planos Gerenciais cadastrados por
essas unidades já serão homologados automaticamente com as opções (Buscar, Cadastrar, Listar e Excluir)
conforme as figuras 02, 03, 04 e 05.

.. figure:: /_static/img/utilizacao/02_Figura_Unidade_Auto-Homologável.png
    :align: center

    Unidade Auto-homologável


.. figure:: /_static/img/utilizacao/03_Figura_Busca_de_Unidade.png
    :align: center

    Busca de Unidade para Cadastro


.. figure:: /_static/img/utilizacao/04_Figura_Cadastro_de_Unidade_Auto-Homologável.png
    :align: center

    Cadastro de Unidade Auto-homologável


.. figure:: /_static/img/utilizacao/05_Figura_Excluir_Unidade_Auto-Homologável.png
    :align: center
    :class: img

    Excluir Unidade Auto-homologável


Plano Gerencial
===============

Funcionalidade que permite cadastrar o Plano Gerencial da Unidade de Localização, que auxiliará a cheﬁa
imediata no planejamento e acompanhamento da unidade, contemplando suas atribuições, processos, atividades e
entregas.

.. figure:: /_static/img/utilizacao/01_Tabela_Opções_na_Tela_do_Plano_Gerencial.png
    :align: center

    Tabela 01: Opções na Tela do Plano Gerencial


**Pendente de Homologação:** Planos Gerenciais da Unidade que foram cadastrados, mas ainda não foram avaliados.

**Necessita de correção: Planos** Gerenciais da Unidade de Localização que não foram homologados pelo Dirigente da Unidade e precisam de correções.

**Homologado:** Planos Gerenciais da Unidade de Localização que foram homologados.

**Finalizado:** Planos Gerenciais da Unidade de Localização com ano de referência expirado.

.. figure:: /_static/img/utilizacao/06_Figura_Fluxo_PG.png
    :align: center

    Fluxo do Plano Gerencial


Cadastro do Plano Gerencial
===========================

Perfil Ativo
------------

Após acessar o sistema caso o servidor possua mais de um vínculo na base de dados do SIGRH será exibido na
tela conforme figura 07.

.. figure:: /_static/img/utilizacao/07_Figura_Vínculos_Ativos_de_Uma_Chefia_de_Unidade.png
    :align: center

    Vínculos Ativos de Uma Chefia de Unidade


Cadstramento de Plano Gerencial
-------------------------------

No menu selecione a funcionalidade do plano gerencial conforme figura 08.

.. figure:: /_static/img/utilizacao/08_Figura_Módulo_do_Plano_Gerencial.png
    :align: center

    Módulo do Plano Gerencial


Novo Plano Gerencial
--------------------

Na tela da figura 09 será possível visualizar e homologar os planos gerenciais das unidades ou criar o plano
da unidade.

.. figure:: /_static/img/utilizacao/09_Figura_Novo_do_Plano_Gerencial.png
    :align: center

    Novo Plano Gerencial

.. note:: O sistema só permite o cadastro de um único plano anualmente.


As Três Etapas Para Criação do Plano Gerencial
----------------------------------------------

Na tela da figura 10 será necessário cadastrar algumas informações para criação do plano.

.. figure:: /_static/img/utilizacao/10_Figura_Identificação_das_Atribuições_da_Unidade.png
    :align: center

    Identificação das Atribuições da Unidade

.. note:: As informações deste exemplo dependem de cada instituição e de cada unidade de uma instituição.

**1ª Etapa:** Identificação das Atribuições da Unidade

**Ano de referência *** 2022

**Atribuições da unidade *** Art.95 da Resolução CONSUP/IFPA nº 191/2020.

.. note::
    Nesta opção pode ser cadastrada mais de uma atribuição conforme características da unidade ou simplesmente
    informar a regulamentação das atribuições da unidade.


**2ª Etapa:** Identificação do Título do Processo de Trabalho

.. figure:: /_static/img/utilizacao/11_Figura_Título_do_Processo_de_Trabalho.png
    :align: center

    Título do Processo de Trabalho

.. note::
    Nesta opção as informações do processo de trabalho podem ser relacionadas com o Plano de Desenvolvimento
    Institucional (PDI) ou outros planos institucionais ou da unidade.

.. note:: As informações deste exemplo dependem de cada instituição e de cada unidade de uma instituição.

Após análise no PDI e em outros planos institucionais pela chefia da unidade 03 (três) **Títulos do Processo
de Trabalho** realizados na unidade foram identificados, sendo 02 (dois) previstos no PDI e 01 (um) previsto
em outro plano da unidade.

**Previsto no PDI:** Novos Módulos do Sistema Integrado de Gestão (SIG) homologados para utilização na
instituição.

**Previsto no PDI:** Projetos Avançados de manutenção, ajustes, novas funcionalidades para os sistemas de
informação em utilização na instituição.

**Não Previsto no PDI:** Suporte dos Sistemas de Informação Utilizados na Instituição.

.. figure:: /_static/img/utilizacao/12_Figura_Processos_de_Trabalho_Cadastrados.png
    :align: center

    Processos de Trabalho Cadastrados


Após o cadastramento de um ou mais processos de trabalho, já será possível submeter o plano a homologação ou
neste momento também poderemos adicionar as atividades do processo.

**3ª Etapa:** Adicionar Atividades ao Processo ou Informações de uma Tabela de Atividade.

.. figure:: /_static/img/utilizacao/13_Figura_Adicionar_Atividades_ao_Processo.png
    :align: center

    Adicionar Atividades ao Processo


.. figure:: /_static/img/utilizacao/14_Figura_Atividades_Adicionadas_aos_Processos_de_Trabalho.png
    :align: center

    Atividades Adicionadas aos Processos de Trabalho


.. figure:: /_static/img/utilizacao/15_Figura_Aviso_Após_Plano_Submetido_para_Homologação.png
    :align: center

    Aviso Após Plano Submetido para Homologação


.. figure:: /_static/img/utilizacao/16_Figura_Homologação_do_Plano_Gerencial.png
    :align: center

    Homologação do Plano Gerencial


Homologação do Plano Gerencial
==============================

Funcionalidade que permite homologar o Plano Gerencial da Unidade de Localização, que auxiliará a cheﬁa
imediata no planejamento e acompanhamento da unidade, contemplando suas atribuições, processos, atividades e
entregas.

.. figure:: /_static/img/utilizacao/02_Tabela_Opções_na_Tela_de_Homologação_do_Plano_Gerencial.png
    :align: center

    Opções na Tela de Homologação do Plano Gerencial


Acesso com o Perfil de Dirigente da Unidade
-------------------------------------------

Após o acesso pelo dirigente da unidade o mesmo deverá acessar com o vínculo da unidade para verificar os
planos pendentes de homologação.

.. figure:: /_static/img/utilizacao/17_Figura_Vínculos_Ativos_de_Um_Dirigente_de_Unidade.png
    :align: center

    Vínculos Ativos de Um Dirigente de Unidade


Planos Pendentes de Homologação
-------------------------------

Nesta tela existem duas opções que podem ser visualizadas (histórico e menu de Ações).

.. figure:: /_static/img/utilizacao/18_Figura_Verificando_Planos_Pendentes_de_Homologação.png
    :align: center

    Verificando Histórico do Plano Pendente de Homologação


Plano Pendente de Homologação
-----------------------------

Utilizando as opções de exibindo histórico e expandir processos.

.. figure:: /_static/img/utilizacao/19_Figura_Avaliar_Plano.png
    :align: center

    Avaliar Plano


.. figure:: /_static/img/utilizacao/20_Figura_Homologar_ou_Justificar.png
    :align: center

    Homologar ou Justificar

.. figure:: /_static/img/utilizacao/21_Figura_Concluir_Homologação.png
    :align: center

    Concluir Homologação


.. figure:: /_static/img/utilizacao/22_Figura_Aviso_de_Avaliação_do_Plano_Gerencial.png
    :align: center

    Aviso de Avaliação do Plano Gerencial


Plano Individual do Servidor
============================

Funcionalidade que permite o servidor cadastrar o seu Plano Individual para cumprimento de suas metas
individuais, onde o mesmo estará vinculado ao Plano Gerencial da Unidade de Localização, contemplando a
relação das atividades do Plano Gerencial com as entregas do Servidor.

.. figure:: /_static/img/utilizacao/03_Tabela_Opções_na_Tela_do_Plano_Gerencial.png
    :align: center

    Opções na Tela do Plano Gerencial


**Pendente de Homologação:** Planos Individuais do Servidor que foram cadastrados, mas ainda não foram avaliados.

**Necessita de correção:** Planos Individuais do Servidor que necessitam de correções.

**Homologado:** Planos Individuais do Servidor que foram homologados pela Chefia Imediata.

**Finalizado:** Planos Individuais do Servidor com ano de referência expirado.

Acesso com o Perfil Servidor
----------------------------

Após o acesso pelo servidor da unidade que só tenha um vínculo não aparece a tela dos vínculos.

.. figure:: /_static/img/utilizacao/23_Figura_Funcionalidades_do_Sistema.png
    :align: center

    Funcionalidades do Sistema


.. note::
    O servidor poderá selecionar a opção para criar o seu Plano Individual ou verificar o Plano Gerencial da
    Unidade.


Plano Individual
----------------

No menu da figura 24 selecione a funcionalidade do plano individual.

.. figure:: /_static/img/utilizacao/24_Figura_Módulo_do_Plano_Individual.png
    :align: center

    Módulo do Plano Individual


Criar Novo Plano Individual
---------------------------

Na tela da figura 25 será possível visualizar e homologar os planos individuais ou criar o plano individual do servidor.

.. figure:: /_static/img/utilizacao/25_Figura_Criar_novo_Plano_Individual.png
    :align: center

    Criar novo Plano Individual


As Duas Etapas Para Criação do Plano Individual
-----------------------------------------------

Na tela da figura 26 será necessário cadastrar algumas informações para criação do plano individual do
servidor.

**1ª Etapa:** Identificação das Atribuições da Unidade

**Nome servidor ***

**Equipe (opcional)**

**Ano ***

**Modalidade de trabalho ***

**Horário de trabalho ***

**Dia da Semana ***

**Horário * (Início * 00:00 Fim 00:00 *)**

.. figure:: /_static/img/utilizacao/26_Figura_Informações_Cadastrais_do_Novo_Plano_Individual.png
    :align: center

    Informações Cadastrais do Novo Plano Individual


.. note::
    Nesta opção pode ser cadastrada mais de um plano individual para o servidor. Com relação ao horário do sistema
    contabiliza a quantidade de horas sem intervalo, ou seja, verifique o limite de carga horária se não é
    superior a praticada pelo servidor.


.. figure:: /_static/img/utilizacao/27_Figura_Carga_Horária_Superior_da_Praticada_Pelo_Servidor.png
    :align: center
    
    Carga Horária Superior da Praticada Pelo Servidor


.. figure:: /_static/img/utilizacao/28_Figura_Cadastro_do_Plano_Individual.png
    :align: center

    Cadastro do Plano Individual


.. figure:: /_static/img/utilizacao/29_Figura_Plano_Individual_Salvo_Com_Sucesso.png
    :align: center

    Plano Individual Salvo Com Sucesso


**2ª Etapa:** Cadastro das Entregas do Plano Individual

.. figure:: /_static/img/utilizacao/30_Figura_Cadastrar_Entregas_do_Plano_Individual.png
    :align: center

    Cadastrar Entregas do Plano Individual


.. note::
    Na tela da fila 31 será cadastrada o título da entrega, que vai ser relacionada com a 3ª etapa do Plano
    Gerencial (Tabela de Atividades).


.. figure:: /_static/img/utilizacao/31_Figura_Vincula_da_Entrega_com_a_Atividade.png
    :align: center

    Vincula da Entrega com a Atividade


.. figure:: /_static/img/utilizacao/32_Figura_Informações_Detalhadas_da_Entrega.png
    :align: center

    Informações Detalhadas da Entrega


.. figure:: /_static/img/utilizacao/33_Figura_Finalizar_Cadastro_da_Entrega.png
    :align: center

    Finalizar Cadastro da Entrega


.. figure:: /_static/img/utilizacao/34_Figura_Cadastrar_Uma_Nova_Entrega.png
    :align: center

    Cadastrar Uma Nova Entrega


.. figure:: /_static/img/utilizacao/35_Figura_Informações_Detalhadas_de_Uma_nova_Entrega.png
    :align: center

    Informações Detalhadas de Uma nova Entrega


.. figure:: /_static/img/utilizacao/36_Figura_Concluir_Entregas.png
    :align: center

    Concluir Entregas


.. figure:: /_static/img/utilizacao/37_Figura_Visualizar_Entregas_Cadastradas.png
    :align: center

    Visualizar Entregas Cadastradas


.. figure:: /_static/img/utilizacao/38_Figura_Status_da_Entrega.png
    :align: center

    Status da Entrega


.. figure:: /_static/img/utilizacao/39_Figura_Cadastro_de_Justificativas.png
    :align: center

    Cadastro de Justificativas


Homologação do Plano Individual do Servidor
===========================================

Funcionalidade que permite homologar o Plano Individual do Servidor.

.. figure:: /_static/img/utilizacao/04_Tabela_Opções_na_Tela_do_Plano_Individual.png
    :align: center

    Opções na Tela do Plano Individual


Acessando com o Perfil Servidor
-------------------------------

Após o acesso pelo servidor da unidade que só tenha um vínculo não aparece a tela dos vínculos.


Relatórios de Entregas
======================

Funcionalidade que permite visualizar os relatórios das entregas dos servidores da instituição. Podendo ser
eles quantitativos que apresentam dados sintetizados referente aos status das entregas, outra forma de
visualização de forma qualitativa que apresenta os dados detalhados das entregas.


.. figure:: /_static/img/utilizacao/40_Figura_Relatório_de_Entregas.png
    :align: center

    Relatório Geral


.. figure:: /_static/img/utilizacao/41_Figura_Relatório_de_Entregas_Quantitativa.png
    :align: center

    Relatório Geral


Referências
===========

Instrução Normativa nº 65, de 30 de julho de 2020
https://www.in.gov.br/en/web/dou/-/instrucao-normativa-n-65-de-30-de-julho-de-2020-269669395

Sistemas e Dados
https://www.gov.br/servidor/pt-br/assuntos/programa-de-gestao/sobre-os-sistemas-propostos

Plataforma de recebimento de dados do Programa de Gestão - PGD
http://hom.api.programadegestao.economia.gov.br/docs

Decreto nº 11.072, de 17 de maio de 2022
https://www.in.gov.br/web/dou/-/decreto-n-11.072-de-17-de-maio-de-2022-401056788

Documentação Negocial Polare STI/UFRN 2022
