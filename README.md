# SAP Fiori Extension

Uma extensão do Chrome para analisar e extrair informações de aplicativos SAP Fiori, incluindo detalhes como objetos semânticos, ações, nomes de CDS, documentos associados e informações SAP.

- **Versão**: 1.0
- **Manifest Version**: 3
- **Autor**: [Seu Nome ou Equipe]
- **Data**: 10 de março de 2025

## Descrição

Esta extensão permite que os usuários analisem páginas de aplicativos SAP Fiori diretamente no navegador Chrome. Ao clicar no botão "Buscar App", a extensão extrai informações como o cliente SAP, objeto semântico, ação, nome da CDS, serviço OData e links para documentações relacionadas. A interface inclui abas para visualizar informações do aplicativo, detalhes SAP e documentações.

## Funcionalidades

- **Detecção de Apps Fiori**: Identifica se a página atual é um aplicativo Fiori com base na URL.
- **Extração de Informações**: Recupera dados como `sap-client`, `semanticObject`, `action`, e nomes de CDS.
- **Interface com Abas**: Exibe informações em três abas: "Informações App", "Informações SAP" e "Documentações".
- **Logotipo Dinâmico**: Carrega o logotipo da empresa diretamente da página Fiori (com fallback para uma URL padrão).
- **Integração com API**: Faz requisições para uma API personalizada para buscar documentos e detalhes SAP.

## Pré-requisitos

- **Navegador**: Google Chrome (versão mais recente recomendada).
- **Permissões**: Acesso às abas ativas e execução de scripts no contexto da página.
- **Conexão à Internet**: Necessária para requisições à API e carregamento de recursos externos.

## API
- **Criar uma Rota na SICF**: `/sap/bc/fioriextension`.
- **Apontar para uma classe que implementa a interface**: `IF_HTTP_EXTENSION`.
- **Implementar o método**: `IF_HTTP_EXTENSION~HANDLE_REQUEST`.
- **Será executado um GET com os parâmetros**:
-   Objeto Semântico: `objeto`
-   Action: `action`
-   Nome: `nome`
-   CDS: `cds`
- **Espera-se um retorno para informações**: Lista de Name/Value `[{ name: 'Informação Exemplo', value: '123' }]`
- **Expera-se um retorno para documentos**: Lista de Name/Value `[{ name: 'Especificação A', value: 'http://google.com.br' }]`

## Exemplo de Código para a Classe
```abap

  METHOD if_http_extension~handle_request.

    " Estrutura de Retorno
    TYPES: BEGIN OF ty_key,
             name  TYPE string,
             value TYPE string,
           END OF ty_key,
           tt_key TYPE TABLE OF ty_key.
    DATA: BEGIN OF ls_response,
            documents TYPE tt_key,
            sap       TYPE tt_key,
          END OF ls_response.

    " Verifica se é um GET
    DATA(lv_type_crud) = server->request->get_header_field( name = '~request_method' ).
    CHECK lv_type_crud = 'GET'.

    " Buscando Entradas
    DATA lt_params TYPE tihttpnvp.
    server->request->get_form_fields( CHANGING fields = lt_params ).
    CHECK lines( lt_params ) > 0.

    DATA(lv_object) = VALUE #( lt_params[ 1 ] OPTIONAL ). " Objeto Semântico
    DATA(lv_action) = VALUE #( lt_params[ 2 ] OPTIONAL ). " Ação
    DATA(lv_nome)   = VALUE #( lt_params[ 3 ] OPTIONAL ). " Nome App
    DATA(lv_cds)    = VALUE #( lt_params[ 4 ] OPTIONAL ). " CDS

    " Exemplo de Documentação
    APPEND VALUE #( name = 'Especificação ABAP' value = 'http://google.com.br' ) TO ls_response-documents.

    " Exemplo de Informações Gerais SAP
    APPEND VALUE #( name = 'RICEFW'       value = '123'           ) TO ls_response-sap.
    APPEND VALUE #( name = lv_object-name value = lv_object-value ) TO ls_response-sap.
    APPEND VALUE #( name = lv_action-name value = lv_action-value ) TO ls_response-sap.
    APPEND VALUE #( name = lv_nome-name   value = lv_nome-value   ) TO ls_response-sap.
    APPEND VALUE #( name = lv_cds-name    value = lv_cds-value    ) TO ls_response-sap.

    " Retorna Valores
    DATA(lv_response_json) = /ui2/cl_json=>serialize( data = ls_response pretty_name =  /ui2/cl_json=>pretty_mode-low_case ).
    server->response->set_cdata( lv_response_json ).
    server->response->set_content_type( `application/json; charset=UTF-8` ).

  ENDMETHOD.

```
