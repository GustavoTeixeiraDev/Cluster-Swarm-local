# Cluster Docker Swarm com Vagrant (4 VMs)

Este projeto cria, via Vagrant + VirtualBox, um **cluster Docker Swarm** com 4 máquinas virtuais:

- `master` (manager)
- `node01` (worker)
- `node02` (worker)
- `node03` (worker)

Cada VM recebe **IP fixo**, todas vêm com **Docker pré-instalado**, e o Swarm é **inicializado automaticamente**, com o `master` como **manager** e os demais ingressando como **workers**.

## 🔧 Pré-requisitos

- [VirtualBox](https://www.virtualbox.org/) (6.x ou 7.x)
- [Vagrant](https://www.vagrantup.com/) (2.3+)
- Acesso a internet para baixar a box `ubuntu/focal64`

> Dica (Windows): execute o `PowerShell` como **Administrador** na primeira vez, pois o VirtualBox pode precisar criar/adaptar interfaces de rede host-only.

## 🗂 Estrutura

├── Vagrantfile
└── README.md

## 🚀 Subindo o ambiente

No diretório do projeto:

vagrant up

O Vagrant irá:

Criar as 4 VMs com IPs privados:

master: 192.168.56.10

node01: 192.168.56.11

node02: 192.168.56.12

node03: 192.168.56.13

Instalar o Docker em todas as VMs.

Executar docker swarm init no master e salvar o token em /vagrant/worker_token.txt.

Fazer cada node ingressar como worker.
✅ Verificando o cluster

Acesse o master:

vagrant ssh master


Liste os nós:

docker node ls


Você deverá ver algo como:

ID                            HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
...                           master    Ready   Active        Leader
...                           node01    Ready   Active
...                           node02    Ready   Active
...                           node03    Ready   Active

📦 Teste rápido (Stack de exemplo)

Do master, faça o deploy de um serviço simples (Nginx):

cat << 'YAML' > demo-stack.yml
version: "3.8"
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
YAML

docker stack deploy -c demo-stack.yml demo
docker stack services demo


Acesse no navegador do host:
http://192.168.56.10:8080

Remover a stack:

docker stack rm demo

🧹 Destruindo o ambiente
# Para parar as VMs
vagrant halt

# Para remover tudo
vagrant destroy -f

🔄 Comandos úteis

Recriar somente uma VM (ex.: node02):

vagrant destroy -f node02
vagrant up node02


Entrar em uma VM específica:

vagrant ssh node01


Atualizar o token de join (caso necessário):

# No master
docker swarm join-token worker
# copie o comando completo exibido e execute na VM worker correspondente

⚙️ Personalização

Memória/CPU: edite no Vagrantfile (bloco config.vm.provider "virtualbox").

IPs: ajuste os IPs no Vagrantfile (rede 192.168.56.0/24 por padrão).

Box base: atualmente ubuntu/focal64. Pode trocar por ubuntu/jammy64 se preferir (ajuste também o repositório Docker no script de instalação, se necessário).

🩺 Troubleshooting

1) “Could not find a host-only adapter” (VirtualBox)
Crie manualmente uma rede Host-Only pelo VirtualBox GUI (ex.: vboxnet0) e ajuste os IPs no Vagrantfile.

2) Token expirado para ingressar workers
No master, gere novamente:

docker swarm join-token worker


E execute o comando exibido nos nós workers.

3) Problema de permissão com Docker no Vagrant
Saia e entre novamente na sessão SSH:

exit
vagrant ssh master


Ou reinicie a VM.

Feito para fins de aprendizado e portfólio.
