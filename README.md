# Instalado Docker no Windows com WSL2

## Instalando o WSL2

Para instalar o WSL2 abra o PowerShell no modo administrador e rode o seguiten comando:

```console
wsl --install
```

Ao final da instalação será necessário reiniciar o computador, ao retornar a aplicação solicitará que seja criado um usuário e senha para o linux.

Mais informações de configurações podem ser consultados no link oficial: https://learn.microsoft.com/pt-br/windows/wsl/install

## Instalando Docker (sem Docker Desktop)

Antes de instalar o Docker Engine pela primeira vez em uma nova máquina host, você precisa configurar o repositório Docker. Depois, você pode instalar e atualizar o Docker a partir do repositório.

```console
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

```

Para instalar a versão mais recente, execute:

```console
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Para validar se a instalação ocorreu com sucesso, rode o comando:

```console
sudo docker run hello-world
```

Se tudo correu certo deve aparacer a seguinte mensagem no seu terminal:

![image](https://github.com/gustavolarsen/configucao-docker-wsl2/assets/55494775/ef28facb-4624-4f41-a338-ff21fb6a99f9)

Mais informações de configurações e versões pode ser consultadas na página ofical em https://docs.docker.com/engine/install/ubuntu/

## Configurando **Dockerfile** para ambiente Angular

Crie um arquivo `Dockerfile` na raiz do projeto conforme o exemplo abaixo:

```dockerfile
# Use a imagem oficial do Node.js como base
FROM node:20

# Instale o Angular CLI globalmente
RUN npm install -g @angular/cli

# Defina o diretório de trabalho no contêiner
WORKDIR /app

# Copie  do código da aplicação para o diretório de trabalho
COPY . .

# Instale as dependências do projeto
RUN npm install

# Exponha a porta em que a aplicação Angular irá rodar
EXPOSE 4200

# Comando para iniciar a aplicação Angular
CMD ["ng", "serve", "--host", "0.0.0.0"]
```

Para criar a imagem com o arquivo Dockerfile temos que rodar os seguinte comando:

Build da imagem
```console
sudo docker build -t <nome-imagem> .
```
Rodar a imagem
```console
sudo docker run -p 4200:4200 <nome-imagem>
```

### Configurando **Dockerfile** para um site estático (build gerada pelo Angular)

Para essa configuraçao será utilizado o [NGINX](https://nginx.org/en/)

```dockerfile
# Use a imagem oficial do Node.js como base
FROM node:20 as builder

# Instale o Angular CLI globalmente
RUN npm install -g @angular/cli

# Defina o diretório de trabalho no contêiner
WORKDIR /app

# Copie  do código da aplicação para o diretório de trabalho
COPY . .

# Instale as dependências do projeto
RUN npm install

# Rodar o comando de build configurado no package.json
RUN npm run build

# Importar o nginx
FROM nginx:alpine

# Copiar o que está no builder pra dentro da pasta que o servidor nginx vai buscar os arquivos estáticos 
COPY --from=builder /app/dist/<nome-aplicao>/browser usr/share/nginx/html

# Copiar as configurações do nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Copiar o arquivo mime.types para que o site consiga carregar arquivos JS
COPY mime.types /etc/nginx/mime.types

# Expor a porta em que a aplicação irá rodar
EXPOSE 80

# Comando para iniciar a aplicação
CMD ["nginx", "-g", "daemon off;"]
```

### Criando o arquivo nginx.conf

```conf
events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;

  server {
    listen 80;
    server_name localhost;

    location / {
      root /usr/share/nginx/html;
       index index.html;
       try_files $uri $uri/ /index.html;
    }

    error_page 500 502 503 504  /50x.html;
    location = /50x.html {
      root /usr/share/nginx/html;
    }
  }
}
```
### Criando o arquivo mime.types

Oesta cofiguração é necessario para que a aplicação consiga 

```types
types {
  text/html  html htm shtml;
  text/css  css;
  application/javascript  js;
  application/json  json;
  image/gif  gif;
  image/jpeg  jpeg jpg;
  application/xml  xml;
  text/xml  xml;
  image/png  png;
}
```

Para criar a imagem com o arquivo Dockerfile usando o nginx.conf devemos rodar o comando:

Build da imagem
```console
sudo docker build -t <nome-imagem> .
```
Rodar a imagem
```console
sudo docker run -p 80:80 <nome-imagem>
```
