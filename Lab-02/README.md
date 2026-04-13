## Lab-02 - Container Technologies

### Exercício 2 - Execução de Contêineres e Volumes

Objetivo: Praticar a execução da mesma imagem criada no Lab-01 e utilizar os principais comandos do dia a dia para contêineres e volumes no Docker.

Neste laboratório, iremos reutilizar a imagem do NGINX criada no Lab-01 para praticar execução, inspeção, acesso ao contêiner e montagem de volumes.

> Observação: para executar este laboratório no GitHub Codespaces, valide antes se o ambiente possui Docker disponível com o comando `docker version`.

### Pré-requisitos

Objetivo: Garantir que a imagem utilizada no laboratório esteja disponível localmente ou publicada no Docker Hub.

Antes de iniciar este laboratório, conclua o Lab-01 e garanta que a imagem abaixo exista no seu ambiente:

```shell
<user>/container-technologies:v1.0.0
```

Caso a imagem não esteja disponível localmente, você pode fazer o pull a partir do Docker Hub:

```shell
docker image pull <user>/container-technologies:v1.0.0
```

### Principais comandos para contêineres

Objetivo: Consultar rapidamente os comandos mais usados na execução e administração de contêineres.

| Comando | Descrição |
|---|---|
| docker container run | Cria e executa um novo contêiner. |
| docker container ps | Lista os contêineres em execução. |
| docker container ps -a | Lista todos os contêineres, inclusive os parados. |
| docker container top | Exibe os processos em execução dentro do contêiner. |
| docker container exec -ti <container-id> /bin/sh | Acessa o shell de um contêiner em execução. |
| docker container logs <container-id> | Exibe os logs do contêiner. |
| docker container stop <container-id> | Interrompe a execução do contêiner. |
| docker container rm -f <container-id> | Remove um contêiner, mesmo que esteja em execução. |

### Principais comandos para volumes

Objetivo: Consultar rapidamente os comandos mais usados para criação e administração de volumes.

| Comando | Descrição |
|---|---|
| docker volume create <volume-name> | Cria um volume nomeado. |
| docker volume ls | Lista os volumes disponíveis. |
| docker volume inspect <volume-name> | Exibe detalhes de um volume. |
| docker container run -v <volume-name>:/caminho/no/container | Executa um contêiner montando um volume. |
| docker volume rm <volume-name> | Remove um volume não utilizado. |
| docker volume prune | Remove volumes não utilizados. |

## Executando a imagem criada no Lab-01

Objetivo: Subir um contêiner com a imagem do Lab-01 e validar que a aplicação está respondendo corretamente.

1. Liste as imagens disponíveis no ambiente:

```shell
docker image ls
```

2. Execute o contêiner em segundo plano com a imagem criada no Lab-01:

```shell
docker container run -d --name lab02-nginx -p 8080:80 <user>/container-technologies:v1.0.0
```

3. Liste os contêineres em execução:

```shell
docker container ps
```

4. No GitHub Codespaces, abra a aba `Ports`.
5. Verifique se a porta `8080` foi encaminhada automaticamente.
6. Clique em `Open in Browser` para abrir a aplicação.
7. Valide que a página exibe a mensagem criada no Lab-01.

## Praticando os principais comandos de contêiner

Objetivo: Explorar os comandos mais usados para listar, inspecionar processos e acessar um contêiner em execução.

1. Liste novamente os contêineres ativos:

```shell
docker container ps
```

2. Liste todos os contêineres, inclusive os que já foram finalizados:

```shell
docker container ps -a
```

3. Exiba os processos em execução dentro do contêiner:

```shell
docker container top lab02-nginx
```

4. Acesse o shell do contêiner:

```shell
docker container exec -ti lab02-nginx /bin/sh
```

5. Dentro do contêiner, valide o conteúdo do arquivo publicado pelo NGINX:

```shell
cat /usr/share/nginx/html/index.html

Container Technologies - LAB 01
```

6. Saia do contêiner com o comando:

```shell
exit
```

## Praticando volumes com a mesma imagem

Objetivo: Montar um volume na mesma imagem do Lab-01 e validar como o conteúdo do volume substitui o conteúdo original da imagem.

1. Crie um volume nomeado:

```shell
docker volume create lab02-html
```

2. Liste os volumes disponíveis:

```shell
docker volume ls
```

3. Inspecione o volume criado:

```shell
docker volume inspect lab02-html
```

4. Preencha o volume com um novo arquivo `index.html` usando um contêiner temporário:

```shell
docker container run --rm \
  -v lab02-html:/volume \
  alpine:latest \
  sh -c 'echo "Volumes - Container Technologies" > /volume/index.html'
```

5. Execute um novo contêiner usando a mesma imagem do Lab-01, agora montando o volume no diretório do NGINX:

```shell
docker container run -d --name lab02-nginx-volume -p 8081:80 \
  -v lab02-html:/usr/share/nginx/html \
  <user>/container-technologies:v1.0.0
```

6. Abra a aba `Ports` no Codespaces e valide a porta `8081`.
7. Clique em `Open in Browser` para abrir a aplicação.
8. Valide que a nova mensagem exibida é:

```text
Volumes - Container Technologies
```

9. Acesse o contêiner com volume e confira o conteúdo:

```shell
docker container exec -ti lab02-nginx-volume /bin/sh
cat /usr/share/nginx/html/index.html

Volumes - Container Technologies
```

10. Saia do contêiner:

```shell
exit
```

## Limpando o ambiente

Objetivo: Remover os contêineres e volumes criados no laboratório para deixar o ambiente pronto para novos testes.

1. Remova os contêineres criados durante o laboratório:

```shell
docker container rm -f lab02-nginx lab02-nginx-volume
```

2. Remova o volume criado:

```shell
docker volume rm lab02-html
```

3. Se desejar remover volumes órfãos, execute:

```shell
docker volume prune
```

4. Para limpar imagens e cache não utilizados, execute:

```shell
docker system prune -a
```

5. Laboratório concluído com sucesso.