# Trabalho Prático sobre Testes Reais de Software

Trabalho para a disciplina de Prática e Desenvolvimento de Softwares na UFMG sobre casos reais de testes de software.

## Repositório observado: Mealie

Para esse estudo, foram analisados os testes unitários e de integração do projeto [Mealie](https://github.com/hay-kot/mealie). O sistema consiste de uma plataforma de gerenciamento de receitas e planejamento de refeições construído utilizando RestAPI e Vue.

Esse projeto foi escolhido por se tratar de um projeto relevante no github (+1k stars), por ter sido desenvolvido na linguagem Vue, linguagem que tenho procurado me aprofundar atualmente, além é claro de possuir cenários de testes sólidos e concretos.

## Ferramenta de Testes

Os testes unitários e de integração do projeto Mealie foram construídos utilizando a estrutura **FastAPI** através da biblioteca [TestClient](https://fastapi.tiangolo.com/tutorial/testing/). Será observado nos exemplos a seguir que, conforme orientado na documentação de testes em FastAPI, as funções de teste iniciam-se com o nome _"test\_"_ para indicar à biblioteca quais os métodos a serem invocados quando for desejado testar a aplicação.

## Teste de Integração

Para os testes de integração, foi selecionado o arquivo [test_settings_routes.py](https://github.com/hay-kot/mealie/blob/dev/tests/integration_tests/test_settings_routes.py) por se tratar do arquivo com a alteração mais recente na pasta de testes de integração. Vale ressaltar que a separação entre pastas de testes de integração e testes unitários auxiliou relevantemente a diferenciação dos dois tipos de testes. Nesse arquivo, vejamos os dois cenários de teste:

```python
import json

import pytest
from fastapi.testclient import TestClient
from mealie.schema.settings import SiteSettings
from tests.app_routes import AppRoutes


@pytest.fixture(scope="function")
def default_settings():
    return SiteSettings().dict(by_alias=True)


def test_default_settings(api_client: TestClient, api_routes: AppRoutes, default_settings):
    response = api_client.get(api_routes.site_settings)

    assert response.status_code == 200

    assert json.loads(response.content) == default_settings


def test_update_settings(api_client: TestClient, api_routes: AppRoutes, default_settings, admin_token):
    default_settings["language"] = "fr-FR"
    default_settings["showRecent"] = False

    response = api_client.put(api_routes.site_settings, json=default_settings, headers=admin_token)

    assert response.status_code == 200

    response = api_client.get(api_routes.site_settings)
    assert json.loads(response.content) == default_settings
```

### 1) test_default_settings

Neste cenário de teste de integração, o método realiza uma requisição GET para uma client api real (portanto, integrando com outra aplicação) buscando as informações de configuração do site. O teste é executado com sucesso caso a resposta da api tenha HTTP Status 200 (OK) e caso o conteúdo da resposta seja o esperado.

### 2) test_update_settings

Já este caso de teste checa se a atualização das configurações está funcionando conforme esperado. Assim como o método anterior, é executada uma requisição real para uma client api, sendo desta vez um PUT que atualiza as configurações padrão do site. O teste é executado com sucesso para HTTP Status 200 (OK) e caso o resultado obtido da requisição GET possua os novos valores atualizados.

## Testes Unitários

O teste unitário escolhido para análise pertence ao arquivo [test_security.py](https://github.com/hay-kot/mealie/blob/dev/tests/unit_tests/test_security.py), referente ao módulo de segurança do sistema. Para facilitar a compreensão, exibe-se abaixo somente o bloco do código a ser explicado, omitindo o restante do arquivo:

```python
from pathlib import Path

from mealie.core import security
from mealie.routes.deps import validate_file_token
from mealie.core.config import settings
from mealie.db.db_setup import create_session


def test_create_file_token():
    file_path = Path(__file__).parent
    file_token = security.create_file_token(file_path)

    assert file_path == validate_file_token(file_token)
```

### 3) test_create_file_token

O teste unitário apresentado tem como objetivo testar a criação de token de arquivo pertencente à estrutura security. Após obter o caminho para o arquivo onde está o módulo (através da constante \_file\_ de Python), o teste invoca a criação de token e, por fim, valida se o token criado pode ser validado através de um método de importado do módulo de rotas do sistema e comparado com o caminho inicial do qual o token foi criado. Caso o token não possa ser criado ou não possa ser validado corretamente após sua criação, então o teste unitário falhará.
