

# Estudo POC - Uso config server

## Abordagens para Organizar Configurações

### 1- Diretórios por Aplicação e Ambiente:
Você pode organizar seu repositório Git para armazenar configurações em diretórios específicos para cada aplicação e ambiente. Por exemplo:

    ├── config-repo
    │   ├── app1
    │   │   ├── application.yml
    │   │   ├── application-dev.yml
    │   │   └── application-prod.yml
    │   └── app2
    │       ├── application.yml
    │       ├── application-dev.yml
    │       └── application-prod.yml

Configuração do Config Server:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/seu-repositorio/config-repo
          searchPaths:
            - app1
            - app2
```

Nesse exemplo, você define searchPaths para os diretórios que contêm as configurações específicas de cada aplicação.

### 2- Configurações por Nome e Perfil:

Outra abordagem é usar a estrutura de nomes de arquivos para distinguir entre diferentes aplicações e perfis:

    ├── config-repo
    │   ├── application-app1-dev.yml
    │   ├── application-app1-prod.yml
    │   ├── application-app2-dev.yml
    │   └── application-app2-prod.yml

Configuração do Config Server:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/seu-repositorio/config-repo

```

Cliente:
Para que o cliente use a configuração correta, você define o nome da aplicação e o perfil:

```yaml
spring:
  application:
    name: app1
  cloud:
    config:
      uri: http://localhost:8888
      label: main
      profile: dev


```
Com isso, o Config Server buscará o arquivo application-app1-dev.yml para a aplicação app1 no perfil dev.


### 3- Uso de Propriedades com Perfil:

Você pode usar o padrão de perfil para buscar configurações específicas de ambiente:

Repositório Git:

    ├── config-repo
    │   ├── application.yml
    │   ├── application-app1.yml
    │   ├── application-app2.yml
    │   ├── application-app1-dev.yml
    │   ├── application-app1-prod.yml
    │   ├── application-app2-dev.yml
    │   └── application-app2-prod.yml

Configuração do Cliente:

```yaml
spring:
  application:
    name: app1
  cloud:
    config:
      uri: http://localhost:8888
      profile: dev

```
Isso permite que você tenha configurações padrão e específicas para cada perfil de ambiente, como dev e prod.


### 4- Hierarquia de Diretórios:

Para um controle mais granular, você pode organizar diretórios e subdiretórios por ambiente e aplicação:

    ├── config-repo
    │   ├── app1
    │   │   ├── dev
    │   │   │   └── application.yml
    │   │   └── prod
    │   │       └── application.yml
    │   └── app2
    │       ├── dev
    │       │   └── application.yml
    │       └── prod
    │           └── application.yml

Configuração do Config Server:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/seu-repositorio/config-repo
          searchPaths:
            - app1
            - app2

```

Cliente:

```yaml
spring:
  application:
    name: app1
  cloud:
    config:
      uri: http://localhost:8888
      profile: dev


```
#### Resumo
- Diretórios Específicos: Organize configurações em diretórios separados para cada aplicação e ambiente.
- Nomes de Arquivo: Use uma nomenclatura que inclua o nome da aplicação e o perfil (por exemplo, application-app1-dev.yml).
- Configurações de Perfil: Utilize perfis e propriedades no application.yml para buscar a configuração correta com base no nome da aplicação e perfil.
- Hierarquia de Diretórios: Estruture seu repositório em uma hierarquia que reflita suas aplicações e ambientes.

## Comportamento de Mesclagem de Configurações

### 1 - Configurações Gerais e Específicas:

- As configurações definidas em application.yml são consideradas as configurações padrão.
- As configurações em arquivos específicos de perfil (como application-dev.yml ou application-prod.yml) substituem ou adicionam às configurações gerais quando o perfil correspondente está ativo.

### 2 - Ordem de Resolução:

- Configuração Global (application.yml): Aplicada a todas as instâncias.
- Configuração Específica de Perfil (application-{profile}.yml): Sobrepõe as configurações gerais quando o perfil está ativo.
- Configuração de Aplicação e Perfil (application-{application}-{profile}.yml): Se você tiver um nome de aplicação e um perfil, o Config Server procurará configurações específicas para essa combinação.


### Exemplo Prático

**Suponha que você tenha a seguinte estrutura no repositório Git:**

	config-repo
    ├── app1
    │   ├── application.yml
    │   ├── application-dev.yml
    │   └── application-prod.yml
    └── app2
    ├── application.yml
    ├── application-dev.yml
    └── application-prod.yml

application.yml (Configuração Global para app1)

    server:
      port: 8080
    logging:
      level:
      root: INFO

application-dev.yml (Configuração Específica de Ambiente para app1 no Perfil dev)

    server:
      port: 8081
    logging:
      level:
      root: DEBUG

application-prod.yml (Configuração Específica de Ambiente para app1 no Perfil prod)

    server:
      port: 8082
    logging:
      level:
      root: WARN

### Configuração do Cliente
Para que o cliente (app1) use a configuração correta, defina o perfil desejado no application.yml do cliente:

application.yml (Cliente app1)
```yaml

spring:
  application:
    name: app1
  cloud:
    config:
      uri: http://localhost:8888
      profile: dev  # ou 'prod', conforme necessário

```
### Comportamento Esperado
- Se o perfil dev está ativo: O Config Server combinará application.yml com application-dev.yml. Portanto, server.port será 8081, e o nível de log será DEBUG.
- Se o perfil prod está ativo: O Config Server combinará application.yml com application-prod.yml. Portanto, server.port será 8082, e o nível de log será WARN.

### Personalização da Mesclagem
Se você precisa de um controle mais refinado sobre a mesclagem e resolução, você pode considerar os seguintes pontos:

- Configurações de Hierarquia: Certifique-se de que os arquivos de configuração estão corretamente organizados e nomeados para refletir a hierarquia e as necessidades de mesclagem.
- Perfil Ativo: Configure perfis no cliente para garantir que o ambiente correto seja aplicado. O Spring Boot usa o perfil ativo para determinar quais configurações específicas devem ser aplicadas.

### Resumo
Mesclagem de Configurações: O Spring Cloud Config Server mescla application.yml com application-{profile}.yml, substituindo ou adicionando configurações específicas de perfil.
Hierarquia de Configurações: Certifique-se de que suas configurações estão corretamente estruturadas e nomeadas para refletir as necessidades de sua aplicação e ambiente.
