## Lab-01 - Docker image

### Objetivo:
Construir imagens Docker, executar contêineres, validar o conteúdo publicado e enviar artefatos para o Docker Hub, incluindo cenários de multi-stage e multi-platform.

Realizar a construção de uma imagem, conhecer alguns comandos comuns e realizar o push da imagem para o Docker Hub.

> Observação: para executar este laboratório no GitHub Codespaces, valide antes se o ambiente possui Docker disponível com o comando `docker version`.

### Diferença entre `docker build` e `docker buildx`

Objetivo: Entender quando usar o build tradicional e quando utilizar um builder avançado com suporte a recursos modernos.

`docker build` é o comando tradicional para gerar uma imagem, normalmente para a plataforma atual do ambiente onde o build está sendo executado.

`docker buildx` usa o BuildKit e permite recursos mais avançados, como build multi-platform, cache otimizado e push direto de manifestos para o registry.

### Instruções que podem ser utilizadas no Dockerfile

Objetivo: Conhecer as instruções mais comuns usadas na definição de imagens Docker.

| **Instrução** | **Descrição** |
|---|---|
| ADD | Adiciona arquivos e diretórios locais ou remotos. |
| ARG | Usa variáveis em tempo de build. |
| CMD | Especifica comandos padrão. |
| COPY | Copia arquivos e diretórios. |
| ENTRYPOINT | Especifica o executável padrão. |
| ENV | Define variáveis de ambiente. |
| EXPOSE | Descreve em quais portas o aplicativo está escutando. |
| FROM | Cria um novo build a partir de uma imagem base. |
| HEALTHCHECK | Verifica a integridade de um contêiner na inicialização. |
| LABEL | Adiciona metadados a uma imagem. |
| MAINTAINER | Especifica o autor da imagem. |
| ONBUILD | Especifica instruções para quando a imagem for usada em um build. |
| RUN | Executa comandos. |
| SHELL | Define o shell padrão de uma imagem. |
| STOPSIGNAL | Especifica o sinal de sistema para finalizar um contêiner. |
| USER | Define o usuário da imagem. |

Dockerfile reference: https://docs.docker.com/reference/dockerfile/

### Comandos que serão utilizados no exercício

Objetivo: Consultar rapidamente os principais comandos usados durante o laboratório.

| Comando | Descrição |
|---|---|
| docker login | Efetuar login no registry. |
| docker build -t nome:tag . | Faz o build da imagem, definindo nome, tag e usando o diretório atual como contexto. |
| docker buildx ls | Lista os builders disponíveis. |
| docker buildx create --use --name multi-builder | Cria e seleciona um builder para builds avançados. |
| docker buildx inspect --bootstrap | Inicializa e valida o builder selecionado. |
| docker buildx build --platform linux/amd64,linux/arm64 -t nome:tag --push . | Faz o build multi-platform e envia a imagem para o registry. |
| docker image ls | Lista as imagens. |
| docker image push | Envia imagens para o registry. |
| docker container ps | Lista os contêineres em execução. |
| docker container ps -a | Lista todos os contêineres, inclusive os parados. |
| docker container run | Executa contêineres. |
| docker container run -p 8080:80 nginx:latest | Executa o contêiner expondo a porta 80 do contêiner na porta 8080 do host. |
| docker container run -p 8080:80 nginx:latest -d | Executa o contêiner expondo a porta 80 do contêiner na porta 8080 do host em segundo plano. |
| docker container rm -f <container-id> | Força a remoção de um ou mais contêineres. |
| docker container exec -ti <container-id> /bin/sh | Acessa um contêiner em execução via /bin/sh. |
| docker system prune --all | Limpa o ambiente, removendo imagens, contêineres parados, cache, volumes e redes fora de uso. |

## Realizando o build de uma imagem Docker no GitHub Codespaces

Objetivo: Criar e validar uma imagem Docker simples no Codespaces, publicando um arquivo HTML com uma mensagem customizada.

1. Crie uma conta no https://hub.docker.com/.
2. Após criar a conta, crie um repositório com o seguinte nome: `container-technologies`.
3. No GitHub, abra o repositório da disciplina/atividade no qual você irá trabalhar.
4. Clique em `Code`.
5. Abra a aba `Codespaces`.
6. Clique em `Create codespace on main`.
7. Aguarde a criação do ambiente. Quando o Codespace terminar de carregar, o VS Code Web será aberto com o terminal integrado disponível.
8. No terminal do Codespace, crie o arquivo `Dockerfile`:

```shell
touch Dockerfile
```

9. Abra o arquivo `Dockerfile` no editor do Codespace e adicione o conteúdo abaixo:

```Dockerfile
FROM nginx:latest
LABEL MAINTAINER="<e-mail impacta>"
COPY index.html /usr/share/nginx/html/
EXPOSE 80
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

10. Salve o arquivo.
11. No terminal, crie o arquivo `index.html`:

```shell
echo "Container Technologies - LAB 01" > index.html
```

12. Realize o build da imagem:

```shell
docker image build -t <user>/container-technologies:v1.0.0 .
```

13. Liste as imagens para confirmar que o build foi realizado:

```shell
docker image ls

REPOSITORY                         TAG       IMAGE ID       CREATED         SIZE
gersontpc/container-technologies   v1.0.0    4506628500ad   4 seconds ago   187MB
```

14. Execute o contêiner em segundo plano:

```shell
docker container run -d -p 8080:80 <user>/container-technologies:v1.0.0

ff27e0f7d6c7a0514a7129f38243b83af566dd1606f06e8eec493f167fa05568
```

15. Liste os contêineres em execução:

```shell
docker container ps

CONTAINER ID   IMAGE                                     COMMAND                  CREATED          STATUS          PORTS                  NAMES
ff27e0f7d6c7   gersontpc/container-technologies:v1.0.0   "nginx -g 'daemon of…"   47 seconds ago   Up 47 seconds   0.0.0.0:8080->80/tcp   interesting_poincare
```

16. No Codespaces, abra a aba `Ports`.
17. Verifique que a porta `8080` foi encaminhada automaticamente. Caso não apareça, adicione a porta manualmente.
18. Clique em `Open in Browser` ou abra a URL gerada para a porta 8080.
19. Valide que o NGINX exibiu a mensagem definida no arquivo `index.html`.
20. Acesse o contêiner em execução:

```shell
docker container exec -ti ff2 /bin/sh
#
```

21. Exiba o conteúdo do arquivo `index.html` dentro do contêiner:

```shell
cat /usr/share/nginx/html/index.html

Container Technologies - LAB 01
```

### Realizando o build com Multi-Stage

Objetivo: Aprender a separar estágios de construção e entrega da imagem, copiando apenas o artefato necessário para a imagem final.

1. Crie um novo arquivo chamado `Dockerfile.multistage`.

```shell
touch Dockerfile.multistage
```

2. Adicione o conteúdo abaixo:

```Dockerfile
FROM alpine:latest AS builder
RUN mkdir -p /app
RUN echo "Multi-Stage Containers Technologies" > /app/index.html

FROM nginx:latest
LABEL MAINTAINER="<e-mail impacta>"
COPY --from=builder /app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

3. Realize o build da imagem multi-stage:

```shell
docker image build -f Dockerfile.multistage -t <user>/container-technologies:v1.0.1 .
```

4. Execute o contêiner com a nova imagem em outra porta do host:

```shell
docker container run -d -p 8081:80 <user>/container-technologies:v1.0.1
```

5. No Codespaces, abra a aba `Ports` e valide a porta `8081`.

6. Abra a aplicação no navegador e confirme a mensagem:

```text
Multi-Stage Containers Technologies
```

7. Acesse o contêiner em execução:

```shell
docker container ps
docker container exec -ti <container-id> /bin/sh
```

8. Valide o conteúdo do arquivo publicado pelo estágio final:

```shell
cat /usr/share/nginx/html/index.html

Multi-Stage Containers Technologies
```

9. Para sair do container execute o atalho `CTRL + D`.

### Realizando o push da imagem para o Docker Hub

Objetivo: Aprender a autenticar no registry e publicar a imagem gerada de maneira prática no Docker Hub.

1. Faça login no [Docker Hub](https://hub.docker.com/):

```shell
docker login
user: <user>
password: <password>

WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

2. Faça o push da imagem para o Docker Hub:

```shell
docker image push <user>/container-technologies:v1.0.0

The push refers to repository [docker.io/gersontpc/container-technologies]
645f56543bbd: Pushed
fd31601f0be4: Mounted from library/nginx
93b4c8c4ac05: Mounted from library/nginx
b7df9f234b50: Mounted from library/nginx
ab75a0b61bd1: Mounted from library/nginx
c1b1bf2f95dc: Mounted from library/nginx
4d99aab1eed4: Mounted from library/nginx
a483da8ab3e9: Mounted from library/nginx
v1.0.0: digest: sha256:a7145b796837b310dda6cae822aae81a3efc86b2b842ac0d00433dbd0a9fa834 size: 1985
```

3. Se desejar, publique também a imagem gerada com multi-stage:

```shell
docker image push <user>/container-technologies:v1.0.1
```

### Realizando o build multi-platform com Buildx

Objetivo: Construir e publicar uma imagem compatível com múltiplas arquiteturas usando `docker buildx`.

1. Liste os builders disponíveis no ambiente:

```shell
docker buildx ls
```

2. Caso necessário, crie um builder e defina-o como padrão da sessão:

```shell
docker buildx create --use --name multi-builder
```

3. Inicialize o builder:

```shell
docker buildx inspect --bootstrap
```

4. Realize o build multi-platform para `linux/amd64` e `linux/arm64`, enviando a imagem diretamente para o mesmo repositório no Docker Hub:

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t <user>/container-technologies:v1.0.2 \
  --push .
```

5. Acompanhe a saída do comando até a publicação do manifesto multi-platform no Docker Hub.

6. Valide no Docker Hub que a mesma imagem/tag foi publicada com suporte a múltiplas arquiteturas.

> Observação: no fluxo com `buildx` e `--push`, a imagem é enviada diretamente ao registry. Por isso, ela pode não aparecer localmente com `docker image ls` como acontece no build tradicional.

### Limpando o ambiente

Objetivo: Remover contêineres, imagens e cache para deixar o ambiente pronto para um novo ciclo de testes.

1. Exclua o contêiner em execução:

```shell
docker container rm -f ff2
```

Se houver um segundo contêiner em execução referente ao teste de multi-stage, remova-o também.

2. Limpe o ambiente:

```shell
docker system prune --all
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: gersontpc/container-technologies:v1.0.0
untagged: gersontpc/container-technologies@sha256:a7145b796837b310dda6cae822aae81a3efc86b2b842ac0d00433dbd0a9fa834
deleted: sha256:4506628500ad9104fb6e01fc19c2979618646918232182662314854d93d3e120

Deleted build cache objects:
o93y66cewb7vlfb3z8lzbz2a7
z9n0isolfvpyzrafm9co816pw
tsk8j3bi9taiupl2h2r3ogvlc
07lge5556g1pgu4l8o71f8f4s
7e7pw7b0kch7oj9iglg8d8azj
ny9r3sozhu25amcj8n74kqhud
ols2afu48w3ho1gziye9amjwv
plxi9anbz76xtrbj2b8u1awvl
8f3f4j960hlze2rqc2xdhjbe9
ivu4avokbermelgxqw0gwg8ut
pqsdihmigklbb1y15hm988b2h

Total reclaimed space: 201B
```

Foi realizada a limpeza das imagens, contêineres parados, cache, redes e demais recursos fora de uso.

### Entregável

Objetivo: Registrar a evidência final do laboratório com o link público da imagem publicada no Docker Hub.

Anexe o link do Docker Hub da imagem após o push no Classroom.

Exemplo: https://hub.docker.com/r/<user>/container-technologies