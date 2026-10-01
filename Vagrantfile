# O Vagrant vai ler este arquivo e entender que precisa criar o seguinte:
# VM 1
# Nome: jenkins
# Hostname: jenkins
# IP: 192.168.56.10
# RAM: 1024 MB
# CPU: 2

# VM 2
# Nome: prod
# Hostname: prod
# IP: 192.168.56.20
# RAM: 1024 MB
# CPU: 1


# Define a versão da configuração utilizada pelo Vagrant
Vagrant.configure("2") do |config|

  # Define a box padrão que será utilizada pelas duas máquinas virtuais
  # ubuntu/jammy64 corresponde ao Ubuntu Server 22.04 LTS
  config.vm.box = "ubuntu/jammy64"


  # ============================================================
  # VM 1 - JENKINS
  # ============================================================

  # Cria uma máquina virtual chamada "jenkins"
  config.vm.define "jenkins" do |jenkins|

    # Define o hostname da máquina dentro do sistema operacional
    jenkins.vm.hostname = "jenkins"
    config.vm.boot_timeout = 600
    # Configura uma rede privada com IP fixo
    # Esse IP será utilizado para acessar a máquina Jenkins
    jenkins.vm.network "private_network",
      ip: "192.168.56.10"
    jenkins.vm.network "forwarded_port", guest: 8080, host: 8050

    # Configura os recursos da máquina virtual no VirtualBox
    jenkins.vm.provider "virtualbox" do |vb|

      # Define o nome exibido da máquina no VirtualBox
      vb.name = "jenkins"

      # Define 1024 MB de memória RAM
      vb.memory = 1024

      # Define 2 CPUs virtuais para o servidor Jenkins
      vb.cpus = 2
    end

    # Executa automaticamente o script responsável
    # pela instalação do Jenkins, Java e Node.js
    # durante o comando "vagrant up"
    jenkins.vm.provision "shell",
      path: "vagrant/scripts/setup-jenkins.sh"
  end


  # ============================================================
  # VM 2 - PRODUÇÃO
  # ============================================================

  # Cria uma segunda máquina virtual chamada "prod"
  config.vm.define "prod" do |prod|

    # Define o hostname do ambiente de produção
    prod.vm.hostname = "prod"

    # Configura um IP privado diferente da VM Jenkins
    prod.vm.network "private_network",
      ip: "192.168.56.20"

    # Configura os recursos da VM de produção no VirtualBox
    prod.vm.provider "virtualbox" do |vb|

      # Define o nome exibido da máquina no VirtualBox
      vb.name = "prod"

      # Define 1024 MB de memória RAM
      vb.memory = 1024

      # Define 1 CPU virtual
      vb.cpus = 1
    end

    # Compartilha a pasta "app" do computador host
    # com a pasta "/home/vagrant/app" dentro da VM de produção
    #
    # Esse recurso permite testar a aplicação Node.js
    # diretamente dentro do ambiente de produção
    prod.vm.synced_folder "./app",
      "/home/vagrant/app"

    # Executa automaticamente o script de configuração
    # do ambiente de produção
    prod.vm.provision "shell",
      path: "vagrant/scripts/setup-prod.sh"
  end

end
