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
