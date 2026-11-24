<h1 align="center"> 🚀 Vagrant </h1>

## 📑 Sumário

- [O que é o Vagrant](#-o-que-é-o-vagrant)
- [Como funciona](#-como-funciona)
- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Primeiros passos](#-primeiros-passos)
- [Comandos essenciais](#-comandos-essenciais)
- [Entendendo o Vagrantfile](#-entendendo-o-vagrantfile)
- [Boxes](#-boxes)
- [Rede](#-rede)
- [Pastas sincronizadas](#-pastas-sincronizadas)
- [Provisionamento](#-provisionamento)
- [Múltiplas máquinas](#-múltiplas-máquinas)
- [Snapshots](#-snapshots)
- [Plugins](#-plugins)
- [Exemplo completo](#-exemplo-completo)
- [Solução de problemas](#-solução-de-problemas)
- [Boas práticas](#-boas-práticas)
- [Referências](#-referências)

---

## 📖 O que é o Vagrant

O [Vagrant](https://www.vagrantup.com/) é uma ferramenta da HashiCorp para criar e gerenciar ambientes de desenvolvimento virtualizados de forma **simples, reproduzível e portátil**.

Em vez de cada pessoa do time configurar a própria máquina "na mão", você descreve o ambiente em um único arquivo (o `Vagrantfile`) e qualquer um sobe o mesmo ambiente com um comando.

**Vantagens:**

- ✅ Ambiente idêntico para todo o time ("na minha máquina funciona" deixa de existir)
- ✅ Configuração versionada junto com o código
- ✅ Ambiente descartável: destrua e recrie quando quiser
- ✅ Isolamento: não "suja" a máquina principal com dependências
- ✅ Onboarding rápido de novos integrantes

---

## ⚙️ Como funciona

O Vagrant não é um virtualizador. Ele trabalha **por cima** de um *provider* (VirtualBox, VMware, Hyper-V, libvirt, Docker...) e automatiza a criação das VMs.

```
Vagrantfile  →  Vagrant  →  Provider (VirtualBox, etc.)  →  Máquina virtual
```

Conceitos principais:

| Conceito | Descrição |
|---|---|
| **Box** | Imagem base do sistema operacional (ex.: Ubuntu, Debian) |
| **Provider** | Software que executa a VM (ex.: VirtualBox) |
| **Provisioner** | Ferramenta que configura a VM após subir (Shell, Ansible, Puppet...) |
| **Vagrantfile** | Arquivo Ruby que descreve toda a configuração |
| **Synced folder** | Pasta compartilhada entre sua máquina e a VM |

---

## 📋 Pré-requisitos

- Um provider instalado. O mais comum é o [VirtualBox](https://www.virtualbox.org/)
- Virtualização habilitada na BIOS/UEFI (VT-x / AMD-V)
- Pelo menos 4 GB de RAM livres (recomendado 8 GB ou mais)
- Espaço em disco para as boxes (cada uma costuma ter alguns GB)

---

## 💾 Instalação

### Windows

Baixe o instalador em [vagrantup.com/downloads](https://developer.hashicorp.com/vagrant/downloads) ou use o gerenciador de pacotes:

```bash
choco install vagrant
# ou
winget install Hashicorp.Vagrant
```

### macOS

```bash
brew install --cask vagrant
```

### Linux (Debian/Ubuntu)

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```

### Verificando a instalação

```bash
vagrant --version
```

---

## 🏁 Primeiros passos

```bash
# 1. Crie uma pasta para o projeto
mkdir meu-projeto && cd meu-projeto

# 2. Gere um Vagrantfile com uma box base
vagrant init ubuntu/jammy64

# 3. Suba a máquina
vagrant up

# 4. Acesse via SSH
vagrant ssh

# 5. Ao terminar, desligue
vagrant halt
```

Para remover tudo (VM e discos):

```bash
vagrant destroy -f
```

---

## 🧰 Comandos essenciais

| Comando | O que faz |
|---|---|
| `vagrant init <box>` | Cria um `Vagrantfile` inicial |
| `vagrant up` | Cria e inicia a máquina |
| `vagrant ssh` | Conecta na VM via SSH |
| `vagrant status` | Mostra o estado da máquina |
| `vagrant halt` | Desliga a VM |
| `vagrant suspend` | Suspende (salva o estado em disco) |
| `vagrant resume` | Retoma uma VM suspensa |
| `vagrant reload` | Reinicia e reaplica o `Vagrantfile` |
| `vagrant reload --provision` | Reinicia e executa o provisionamento novamente |
| `vagrant provision` | Executa o provisionamento sem reiniciar |
| `vagrant destroy` | Remove a VM |
| `vagrant global-status` | Lista todas as VMs do Vagrant na máquina |
| `vagrant validate` | Valida a sintaxe do `Vagrantfile` |
| `vagrant ssh-config` | Exibe a configuração SSH da VM |
| `vagrant port` | Lista as portas redirecionadas |
| `vagrant box list` | Lista as boxes baixadas |
| `vagrant box add <box>` | Baixa uma box |
| `vagrant box remove <box>` | Remove uma box |
| `vagrant box update` | Atualiza a box do projeto |
| `vagrant snapshot ...` | Gerencia snapshots |
| `vagrant plugin ...` | Gerencia plugins |

> 💡 Use `vagrant <comando> --help` para ver as opções de qualquer comando.

---

## 📝 Entendendo o Vagrantfile

O `Vagrantfile` é escrito em Ruby, mas você não precisa saber Ruby para usá-lo.

```ruby
Vagrant.configure("2") do |config|
  # Box base
  config.vm.box = "ubuntu/jammy64"

  # Nome do host
  config.vm.hostname = "dev-box"

  # Configurações do provider
  config.vm.provider "virtualbox" do |vb|
    vb.name   = "dev-box"
    vb.memory = 2048
    vb.cpus   = 2
  end
end
```

Principais opções:

| Opção | Exemplo | Descrição |
|---|---|---|
| `config.vm.box` | `"ubuntu/jammy64"` | Box utilizada |
| `config.vm.box_version` | `"20240101.0.0"` | Fixa a versão da box |
| `config.vm.hostname` | `"dev-box"` | Hostname da VM |
| `config.vm.boot_timeout` | `600` | Tempo máximo de boot (segundos) |
| `config.ssh.forward_agent` | `true` | Encaminha o agente SSH |

---

## 📦 Boxes

As boxes são imagens prontas de sistemas operacionais. O catálogo público fica em [portal.cloud.hashicorp.com/vagrant/discover](https://portal.cloud.hashicorp.com/vagrant/discover).

```bash
# Baixar uma box
vagrant box add ubuntu/jammy64

# Listar boxes locais
vagrant box list

# Ver boxes desatualizadas
vagrant box outdated

# Atualizar
vagrant box update

# Remover
vagrant box remove ubuntu/jammy64

# Limpar versões antigas
vagrant box prune
```

Algumas boxes populares:

- `ubuntu/jammy64` (Ubuntu 22.04)
- `ubuntu/focal64` (Ubuntu 20.04)
- `debian/bookworm64` (Debian 12)
- `generic/rocky9`, `generic/alma9`
- `bento/ubuntu-22.04`

> ⚠️ A disponibilidade de boxes varia conforme o provider e a arquitetura (x86_64 ou ARM). Confira a compatibilidade antes de usar.

---

## 🌐 Rede

### Redirecionamento de portas (port forwarding)

Acessa um serviço da VM pelo `localhost` da sua máquina:

```ruby
config.vm.network "forwarded_port", guest: 80, host: 8080
```

Agora `http://localhost:8080` abre o que roda na porta 80 da VM.

### Rede privada (IP fixo)

Cria uma rede apenas entre sua máquina e a VM:

```ruby
config.vm.network "private_network", ip: "192.168.56.10"
```

### Rede pública (bridge)

A VM recebe um IP na mesma rede local da sua máquina:

```ruby
config.vm.network "public_network"
```

| Tipo | Acesso externo | Uso típico |
|---|---|---|
| `forwarded_port` | Via `localhost` | Acessar um serviço rapidamente |
| `private_network` | Somente host ↔ VM | Ambientes de desenvolvimento e multi-VM |
| `public_network` | Toda a rede local | Expor a VM para outros dispositivos |

---

## 📂 Pastas sincronizadas

Por padrão, a pasta do projeto é montada em `/vagrant` dentro da VM.

```ruby
# Pasta extra
config.vm.synced_folder "./app", "/var/www/app"

# Desabilitar a pasta padrão
config.vm.synced_folder ".", "/vagrant", disabled: true

# Usando rsync (mais rápido em alguns cenários)
config.vm.synced_folder "./app", "/var/www/app", type: "rsync",
  rsync__exclude: [".git/", "node_modules/"]
```

Tipos comuns: `virtualbox`, `nfs`, `rsync`, `smb`.

---

## 🛠️ Provisionamento

O provisionamento automatiza a instalação e configuração do ambiente na primeira vez que a VM sobe (ou quando você executa `vagrant provision`).

### Shell inline

```ruby
config.vm.provision "shell", inline: <<-SHELL
  apt-get update
  apt-get install -y nginx
SHELL
```

### Script externo

```ruby
config.vm.provision "shell", path: "scripts/bootstrap.sh"
```

### Script executado como usuário comum

```ruby
config.vm.provision "shell", privileged: false, inline: <<-SHELL
  echo "Rodando como o usuário vagrant"
SHELL
```

### Outros provisioners

- **Ansible** (`ansible` / `ansible_local`)
- **Puppet**
- **Chef**
- **Docker**
- **File** (copia arquivos para a VM)

```ruby
config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
```

### Controlando quando executar

```ruby
config.vm.provision "shell", inline: "echo sempre", run: "always"
config.vm.provision "shell", inline: "echo uma vez", run: "once"
config.vm.provision "shell", inline: "echo manual", run: "never"
```

```bash
# Executar um provisioner específico
vagrant provision --provision-with nome-do-provisioner
```

---

## 🖥️ Múltiplas máquinas

Dá para subir um ambiente completo (ex.: aplicação + banco) com várias VMs no mesmo `Vagrantfile`:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "web" do |web|
    web.vm.hostname = "web"
    web.vm.network "private_network", ip: "192.168.56.11"
  end

  config.vm.define "db" do |db|
    db.vm.hostname = "db"
    db.vm.network "private_network", ip: "192.168.56.12"
  end
end
```

```bash
vagrant up          # sobe todas
vagrant up web      # sobe apenas a "web"
vagrant ssh db      # acessa a "db"
vagrant destroy -f  # remove todas
```

---

## 📸 Snapshots

Salve o estado da VM para voltar a ele depois (ótimo para testar mudanças arriscadas):

```bash
vagrant snapshot save estado-limpo
vagrant snapshot list
vagrant snapshot restore estado-limpo
vagrant snapshot delete estado-limpo
```

---

## 🔌 Plugins

```bash
vagrant plugin install vagrant-vbguest   # mantém o VirtualBox Guest Additions atualizado
vagrant plugin install vagrant-hostmanager
vagrant plugin list
vagrant plugin update
vagrant plugin uninstall vagrant-vbguest
```

---

## 🧪 Exemplo completo

Um `Vagrantfile` pronto para um ambiente de desenvolvimento Node.js:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "node-dev"

  # Rede
  config.vm.network "forwarded_port", guest: 3000, host: 3000
  config.vm.network "private_network", ip: "192.168.56.20"

  # Pasta do projeto
  config.vm.synced_folder ".", "/vagrant"

  # Recursos
  config.vm.provider "virtualbox" do |vb|
    vb.name   = "node-dev"
    vb.memory = 2048
    vb.cpus   = 2
  end

  # Provisionamento
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y curl git build-essential
    curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
    apt-get install -y nodejs
    node -v && npm -v
  SHELL
end
```

Uso:

```bash
vagrant up
vagrant ssh
cd /vagrant
npm install && npm start
```

---

## 🩺 Solução de problemas

| Problema | Possível solução |
|---|---|
| `vagrant up` trava em "Waiting for machine to boot" | Verifique se a virtualização está ativa na BIOS e aumente `config.vm.boot_timeout` |
| Erro de provider não encontrado | Confirme se o VirtualBox (ou outro provider) está instalado e compatível com a versão do Vagrant |
| Conflito no VirtualBox com Hyper-V (Windows) | Desative o Hyper-V ou use o provider `hyperv` |
| Porta já em uso | Altere o `host:` do `forwarded_port` ou use `auto_correct: true` |
| Pasta sincronizada não aparece | Reinstale o plugin `vagrant-vbguest` ou use `type: "rsync"` / `nfs` |
| Provisionamento não executou novamente | Use `vagrant provision` ou `vagrant reload --provision` |
| Box desatualizada | `vagrant box update` |
| `Vagrantfile` com erro | Rode `vagrant validate` |
| VM "fantasma" ou estado inconsistente | `vagrant destroy -f` e `vagrant up` novamente |

Para ver logs detalhados:

```bash
VAGRANT_LOG=info vagrant up
```

No Windows (PowerShell):

```powershell
$env:VAGRANT_LOG="info"; vagrant up
```

---

## ✅ Boas práticas

- 📌 **Fixe a versão da box** (`config.vm.box_version`) para garantir reprodutibilidade
- 🔒 Versione o `Vagrantfile` no Git, mas **não** versione a pasta `.vagrant/`
- 🧹 Adicione `.vagrant/` ao `.gitignore`
- 🧩 Mantenha o provisionamento **idempotente** (rodar duas vezes não pode quebrar nada)
- 📁 Separe scripts longos em arquivos dentro de uma pasta `scripts/`
- 🔑 Nunca coloque senhas ou tokens no `Vagrantfile`; use variáveis de ambiente
- 💾 Use snapshots antes de mudanças grandes
- 🗑️ Destrua VMs que não usa mais para liberar disco

---

## 📚 Referências

- [Documentação oficial do Vagrant](https://developer.hashicorp.com/vagrant/docs)
- [Catálogo de boxes](https://portal.cloud.hashicorp.com/vagrant/discover)
- [Documentação do VirtualBox](https://www.virtualbox.org/manual/)

---

## Contatos

[![Github Badge](https://img.shields.io/badge/-Github-000?style=flat-square&logo=Github&logoColor=white&link=https://github.com/cleibp)](https://github.com/cleibp)
[![Linkedin Badge](https://img.shields.io/badge/-LinkedIn-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/cleitonpaiva/)](https://www.linkedin.com/in/cleitonpaiva/)
[![Whatsapp Badge](https://img.shields.io/badge/-Whatsapp-4CA143?style=flat-square&labelColor=4CA143&logo=whatsapp&logoColor=white&link=https://api.whatsapp.com/send?phone=5516988368457&text=Ola!)](https://api.whatsapp.com/send?phone=5516988368457&text=Ola!)
[![Gmail Badge](https://img.shields.io/badge/-Gmail-c14438?style=flat-square&logo=Gmail&logoColor=white&link=mailto:cleibp@gmail.com)](mailto:cleibp@gmail.com)

[Pague-me um ☕](https://www.buymeacoffee.com/cleibp)

Feito com muito ❤️☕👨🏻‍💻 por Cleiton Paiva
