# Full Cycle - Desafio 03 - Processo de CI

Nessa fase, você deverá repetir o processo ensinado no módulo.

Pré requisitos:

- Instalar o SDK do Google Cloud que está na URL: https://cloud.google.com/sdk/install

- Criar um novo docker-compose-gcp.yaml

- Criar um novo arquivo chamado cloudbuild.yaml (Que será usado pelo SDK do Google.)

  - Tem mais funcionalidades que um Dockerfile

- Instalar o docker-compose dentro do Registry do google

  - Criar novo Dockerfile e cloudbuild.yaml para gerar imagem com o docker-compose

    - Copiar os arquivos Dockerfile e cloudbuild.yaml do repositório [cloud-builders-community](https://github.com/GoogleCloudPlatform/cloud-builders-community/tree/master/docker-compose) 

    - Gerar um novo [repositório no Github](https://github.com/phmelo/fullcycle-desafio-ci-gcp-docker-compose), contendo os arquivos. 

    - Entrar no Google Cloud Platform

      - Entrar na sua aplicação.

      - Entrar no Google Cloud Build (Clicando no menu lateral no topo a esquerda. Em Tools)

        - Depois clique em Triggers ("Acionadores") e siga os passos da interface
          - Connect Repository (Github)
            - Ele vai pedir para autenticar e instalar o Google Cloud Build no Github.
          - Selecione o repositório do github
            - Criar push trigger (Depois edite)
            - Em build configuration, troque para o arquivo cloudbuild.yaml
            - Após finalizar a criação da Trigger, clique em "Run Trigger"

      - Depois de terminar o processo de build ela estará em Container Registry, pronta para ser usada.

        

- Criar nova Trigger para gerar novas imagens da aplicação Laravel com a imagem docker-compose.

  - Usar Docker do cloud-builders para buildar imagem do docker-compose
  - PUSH da imagem do docker-compose no cgr.io/$PROJECT_ID





1) Você deverá pegar sua aplicação Laravel das fases anteriores e  adicioná-la em um pipeline de integração contínua utilizando o Google Cloud Build, para isso terá que:

- Gerar a imagem do docker-compose e fazer o push no seu Container Registry do GCP. -OK
- Criar uma trigger para ser disparada todas as vezes que um commit entrar no repositório do Github.
- Os steps do Google Cloud Build deverão ser: 
  - Executar o docker-compose
  - Executar o composer
  - Copiar o arquivo .env.example para .env
  - Rodar um artisan key:generate
  - Executar as migrações
  - Executar os testes utilizando o PHPUnit



2) Você deverá instalar a App do Google Cloud Build disponível no  Market Place do Github. Crie um branch develop em seu repositório. Todas as vezes que uma pull request for criada, imediatamente o Google Cloud  Build deverá iniciar o processo de CI.





Bonus:

https://cloud.google.com/sdk/install



**1)** Utilize o sistema de templates do Dockerize para deixar o arquivo **nginx.conf** mais flexível, ou seja, tanto o **host** e **porta** da chamada possam ser definidos como variáveis de ambiente no docker-compose.yaml. 



Solução:

- Criar um arquivo de template do NGINX

```nginx
./.docker/nginx/nginx.conf.tmpl
server {
    listen 80;
    index index.php index.html;
    root /var/www/public;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass {{.Env.APP_HOST}}:{{.Env.APP_PORT}};
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}

```



- Adicionar ao Dockerfile do NGINX a instalação do Dockerize
- Copiar o arquivo de template para o Container do NGINX
- Remover o arquivo de configuração padrão do NGINX

```dockerfile
./.docker/nginx/Dockerfile
FROM nginx:1.15.0-alpine

RUN apk add --no-cache openssl

ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz
    
RUN rm /etc/nginx/conf.d/default.conf
COPY ./nginx.conf.tmpl /etc/nginx/conf.d/

```



- Alterar o arquivo docker-compose para utilizar o dockerize para criar o novo arquivo nginx.conf a partir do template nginx.conf.tmpl
  - Atenção no "entrypoint" e no "command", sem o command o nginx não iniciou.

```yaml
./docker-compose.yaml
...
nginx:
    build: .docker/nginx
    container_name: nginx
    entrypoint: dockerize -template /etc/nginx/conf.d/nginx.conf.tmpl:/etc/nginx/conf.d/nginx.conf nginx
    command: -g "daemon off;" -c /etc/nginx/nginx.conf
    environment:
        - APP_HOST=app
        - APP_PORT=9000
    restart: always
...

```





**2)** Você terá que publicar uma imagem no docker hub. Quando executarmos:
docker run <seu-user>/codeeducation 
Temos que ter o seguinte resultado: Code.education Rocks!

**3)** A imagem de nosso projeto Go precisa ter menos de 2MB =)

Solução:

- Instalar o Golang

```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install golang-go -y
```

- Criar um hello-world.go

```go
package main
import "fmt"
func main() {
  fmt.Printf("Code.education Rocks!\n")
}
```

- Criar uma imagem docker com o código 

```dockerfile
FROM golang:1.13

WORKDIR /go/src/app
COPY ./hello-world.go .

RUN go build ./hello-world.go
RUN ./hello-world

ENTRYPOINT ["./hello-world"]
```



- Criar uma nova imagem menor, usando multistaging build

```dockerfile
FROM golang:alpine as builder

WORKDIR /app
COPY hello-world.go .
RUN GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" -o /app/hello-world .

FROM scratch
COPY --from=builder /app/hello-world .

ENTRYPOINT ["/hello-world"]
```



A única forma dessa imagem ficar com menos de 2MB foi utilizando o comando:

```dockerfile
RUN GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" -o /app/hello-world .
```

A tentativa com o "alpine:3.11" ficou com ~7MB e com o SCRATCH sem o comando acima ficou com ~2MB.



Para baixar e rodar o container no Docker Hub: 

```
docker run phmelomorais/codeeducation
```



