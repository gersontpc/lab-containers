## Lab-03 - Docker Compose (WordPress)

## Objetivo:
Praticar a criação e execução de uma aplicação com múltiplos serviços usando Docker Compose no GitHub Codespaces, validando rede, volumes, variáveis de ambiente e gerenciamento do ciclo de vida dos contêineres.

Neste laboratório, iremos subir uma aplicação WordPress com banco de dados MariaDB usando Docker Compose.

> Observação: para executar este laboratório no GitHub Codespaces, valide antes se o ambiente possui Docker disponível com o comando `docker version`.

### O que é Docker Compose

Objetivo: Entender o papel do Docker Compose na definição e execução de aplicações compostas por múltiplos contêineres.

O Docker Compose permite definir serviços, redes, volumes e variáveis em um único arquivo YAML, facilitando a execução e administração de ambientes com mais de um contêiner.

### Comandos que serão utilizados no exercício

Objetivo: Consultar rapidamente os principais comandos usados durante o laboratório.

| Comando | Descrição |
|---|---|
| docker compose up -d | Cria e inicia os serviços em segundo plano. |
| docker compose ps | Lista os serviços e contêineres gerenciados pelo Compose. |
| docker compose logs -f | Acompanha os logs dos serviços em tempo real. |
| docker compose exec wordpress bash | Acessa o contêiner do WordPress. |
| docker compose config | Exibe a configuração final resolvida do Compose. |
| docker compose down | Remove os contêineres e a rede criada pelo Compose. |
| docker compose down -v | Remove os contêineres, a rede e os volumes nomeados. |
| docker system prune -a | Remove imagens não utilizadas, cache e recursos parados. |

## Preparando o ambiente no GitHub Codespaces

Objetivo: Criar ou acessar um Codespace pronto para executar Docker Compose sem depender de Cloud9 ou EC2.

1. No GitHub, abra o repositório da disciplina ou este repositório.
2. Clique em `Code`.
3. Abra a aba `Codespaces`.
4. Clique em `Create codespace on main`.
5. Aguarde a abertura do VS Code Web.
6. No terminal do Codespace, valide o Docker e o Compose:

```shell
docker version
docker compose version
```

7. Caso esteja utilizando outro repositório da disciplina, clone este conteúdo no terminal:

```shell
git clone https://github.com/gersontpc/lab-containers.git
```

## Estrutura do laboratório

Objetivo: Identificar os arquivos usados para subir a aplicação com Docker Compose.

1. Acesse o diretório do laboratório:

```shell
cd Lab-03/
```

2. Exiba o conteúdo do arquivo `compose.yaml`:

```shell
cat compose.yaml
```

Conteúdo do `compose.yaml`:

```yaml
services:
  mysql:
    image: mariadb:latest
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: senha1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: UserBlog
      MYSQL_PASSWORD: PwdBlog
    expose:
      - "3306"
    volumes:
      - database:/var/lib/mysql
    networks:
      - wordpress

  wordpress:
    image: wordpress:latest
    restart: always
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: UserBlog
      WORDPRESS_DB_PASSWORD: PwdBlog
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
    ports:
      - "8080:80"
    networks:
      - wordpress
    depends_on:
      - mysql

volumes:
  database:
  wordpress:

networks:
  wordpress:
    driver: bridge
```

## Subindo a aplicação WordPress com Docker Compose

Objetivo: Iniciar a aplicação completa com banco de dados e frontend web usando um único comando.

1. No diretório `Lab-03`, execute:

```shell
docker compose up -d
```

2. Verifique os serviços em execução:

```shell
docker compose ps
```

3. Caso queira validar a configuração final interpretada pelo Compose:

```shell
docker compose config
```

4. No Codespaces, abra a aba `Ports`.
5. Verifique se a porta `8080` foi encaminhada automaticamente. Caso não apareça, adicione a porta manualmente.
6. Clique em `Open in Browser` para abrir a aplicação WordPress.

## Configurando o WordPress

Objetivo: Concluir a instalação inicial da aplicação e validar a integração entre WordPress e MariaDB.

1. Ao abrir a aplicação, selecione `Português do Brasil` e clique em `Continuar`.
2. Preencha a tela inicial com os dados abaixo:

- **Título do site:** container-technologies
- **Nome do usuário:** wpuser
- **Senha:** defina uma senha segura
- **O seu e-mail:** seu e-mail institucional

3. Clique em `Instalar WordPress`.
4. Na tela de login, entre com o usuário `wpuser` e a senha definida.
5. Ao acessar o painel de administração, clique no nome do site no canto superior esquerdo para visualizar o frontend.

## Inspecionando os serviços do Compose

Objetivo: Verificar o estado dos contêineres e acessar logs e shell dos serviços.

1. Liste novamente os serviços:

```shell
docker compose ps
```

2. Acompanhe os logs do WordPress:

```shell
docker compose logs -f wordpress
```

3. Em outro terminal, acesse o contêiner do WordPress:

```shell
docker compose exec wordpress bash
```

4. Caso queira validar o banco de dados, acompanhe os logs do MariaDB:

```shell
docker compose logs -f mysql
```

## Definindo limites de recursos dos contêineres

Objetivo: Subir os serviços com restrições básicas de CPU e memória para praticar controle de consumo de recursos.

> Observação: no Docker Compose local, o uso de limites funciona de forma diferente do Docker Swarm. Neste laboratório, o arquivo foi adaptado para um cenário prático de Compose no Codespaces.

1. Exiba o conteúdo do arquivo `compose-limits.yml`:

```shell
cat compose-limits.yml
```

Conteúdo do `compose-limits.yml`:

```yaml
services:
  mysql:
    image: mariadb:latest
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: senha1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: UserBlog
      MYSQL_PASSWORD: PwdBlog
    expose:
      - "3306"
    volumes:
      - database:/var/lib/mysql
    networks:
      - wordpress
    cpus: 1.5
    mem_limit: 1024m

  wordpress:
    image: wordpress:latest
    restart: always
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: UserBlog
      WORDPRESS_DB_PASSWORD: PwdBlog
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
    ports:
      - "8080:80"
    networks:
      - wordpress
    depends_on:
      - mysql
    cpus: 1.0
    mem_limit: 512m

volumes:
  database:
  wordpress:

networks:
  wordpress:
    driver: bridge
```

2. Recrie os serviços com o arquivo de limites:

```shell
docker compose down
docker compose -f compose-limits.yml up -d
```

3. Liste novamente os contêineres:

```shell
docker compose -f compose-limits.yml ps
```

4. Se quiser observar o consumo em tempo real, execute:

```shell
docker stats
```

## Limpando o ambiente

Objetivo: Remover contêineres, rede, volumes e imagens não utilizadas para deixar o ambiente pronto para novos testes.

1. Remova o ambiente criado com o arquivo de limites:

```shell
docker compose -f compose-limits.yml down -v
```

2. Limpe recursos não utilizados do Docker:

```shell
docker system prune -a
```

Output esperado: pressione `y` para confirmar.

```output
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y
```

3. Laboratório concluído com sucesso.