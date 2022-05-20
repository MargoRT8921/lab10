# lab10
Vagrant это менеджер ваших виртуальных окружений для разработки. Фактически он является надстройкой над программой VirtualBox от Oracle, и обеспечивает быстрое создание и настройку виртуальных машин. Созданные таким образом виртуальные машины - боксы (boxes) используются разработчиками web-приложений для построения необходимой среды разработки. Затем они могут быть упакованы в специальные контейнеры (боксы), для установки и использования другими разработчиками в команде.

1 Устанавливаем значение переменных окружения 
2 Указываем имя пользователя Github 
3 Указываем используемый пакетный менеджер (apt)

$ export GITHUB_USERNAME=<имя_пользователя>
$ export PACKAGE_MANAGER=<пакетный_менеджер>
4 Переходим в рабочую директорию 5 Устанавливаем vagrant

$ cd ${GITHUB_USERNAME}/workspace
$ ${PACKAGE_MANAGER} install vagrant
6 Выводим версию скаченного vagrant 
7 Создаем новую виртуальную машину 
8 Выводим содержимое Vagrantfile

$ vagrant version
$ vagrant init bento/ubuntu-19.10
$ less Vagrantfile
$ vagrant init -f -m bento/ubuntu-19.10
9 Создаем директорию shared

$ mkdir shared
10 В файл Vagrantfile записываем комманды для запуска скрипта

$ cat > Vagrantfile <<EOF
\$script = <<-SCRIPT
sudo apt install docker.io -y
sudo docker pull fastide/ubuntu:19.04
sudo docker create -ti --name fastide fastide/ubuntu:19.04 bash
sudo docker cp fastide:/home/developer /home/
sudo useradd developer
sudo usermod -aG sudo developer
echo "developer:developer" | sudo chpasswd
sudo chown -R developer /home/developer
SCRIPT
EOF
11 В файл Vagrantfile записываем конфигурацию для виртуальной машины vagrant-vbguest - это плагин, который автоматически обновляет гостевые дополнения VirtualBox

$ cat >> Vagrantfile <<EOF

Vagrant.configure("2") do |config|

  config.vagrant.plugins = ["vagrant-vbguest"]
EOF
12 Продолжаем конфигурацию для виртуальной машины

$ cat >> Vagrantfile <<EOF

  config.vm.box = "bento/ubuntu-19.10"
  config.vm.network "public_network"
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: \$script, privileged: true

  config.ssh.extra_args = "-tt"

end
EOF
13 Выполняем команду для устронения неполадок "vagrant plugin install vagrant-vbguest" 
  14 vagrant validate - проверка корректности файла Vagrantfile 
  15 Просмотрим список вируальных машин и их статусы 
  16 Запуск виртуальной машины 
  17 информация о проброске портов 
  18 подключение по ssh к запущенной виртуальной машине 
  19 посмотреть список сохранённых состояний виртальной машины 
  20 остановить виртуальную машину 
  21 востановить состояние виртуальной машины по снимку

$ vagrant validate

$ vagrant status
$ vagrant up # --provider virtualbox
$ vagrant port
$ vagrant status
$ vagrant ssh

$ vagrant snapshot list
$ vagrant snapshot push
$ vagrant snapshot list
$ vagrant halt
$ vagrant snapshot pop
22 настраиваем Vagrant для работы с VMware

  config.vm.provider :vmware_esxi do |esxi|

    esxi.esxi_hostname = '<exsi_hostname>'
    esxi.esxi_username = 'root'
    esxi.esxi_password = 'prompt:'

    esxi.esxi_hostport = 22

    esxi.guest_name = '${GITHUB_USERNAME}'

    esxi.guest_username = 'vagrant'
    esxi.guest_memsize = '2048'
    esxi.guest_numvcpus = '2'
    esxi.guest_disk_type = 'thin'
  end
$ vagrant plugin install vagrant-vmware-esxi
$ vagrant plugin list
$ vagrant up --provider=vmware_esxi
