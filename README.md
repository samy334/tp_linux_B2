# TP2 : Déploiement automatisé

## I. Déploiement simple

Créer un Vagrantfile qui :

-   utilise la box centos/7
    
-   crée une seule VM
    
    -   1Go RAM
    -   ajout d'une IP statique  `192.168.2.11/24`
    -   définition d'un nom (interne à Vagrant)
    -   définition d'un hostname


   

>      Vagrant.configure("2")do|config|
>       node1_DISK='./node1_DISK.vdi'
>       config.vm.box="centos/7"
>       config.vbguest.auto_update = false
>       config.vm.box_check_update = false  
>       config.vm.synced_folder ".", "/vagrant", disabled: true
>     
>       config.vm.define "vm_centos7" do |node1|
>     
>         node1.vm.hostname = "centos-tp"
>         node1.vm.network "private_network", ip: "192.168.2.11", netmask: "255.255.255.0"
>         node1.vm.provider "virtualbox" do |vb|
>           vb.customize ["modifyvm", :id, "--memory", "1024"]
>     
>           unless File.exist?(node1_DISK)
>             vb.customize ['createhd', '--filename', node1_DISK, '--variant', 'Fixed', '--size', 5 * 1024]
>           end
>           # Attache le disque à la VM
>             vb.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium',
> node1_DISK]
>         end
>         node1.vm.provision "shell", path: "script.sh"
>       end
>     
>     end


Modifier le Vagrantfile

- la machine exécute un script shell au démarrage qui install le paquet vim

    > node1.vm.provision "shell", path: "script.sh"

Contenu de script.sh

    yum install -y vim

- ajout d'un deuxième disque de 5Go à la VM à la première ligne

 

       unless File.exist?(node1_DISK)
            vb.customize ['createhd', '--filename', node1_DISK, '--variant', 'Fixed', '--size', 5 * 1024]
          end
          # Attache le disque à la VM
            vb.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', node1_DISK]
        end


## III. Multi-node deployment

Créer un Vagrantfile qui lance deux machines virtuelles, les VMs DOIVENT utiliser votre box repackagée comme base


Nouveau Vagrantfile :

```
Vagrant.configure("2")do|config|
    config.vm.box="b2-tp2-centos"

    # Ajoutez cette ligne afin d'accélérer le démarrage de la VM (si une erreur 'vbguest' est levée, voir la note un peu plus bas)
    config.vbguest.auto_update = false

    # Désactive les updates auto qui peuvent ralentir le lancement de la machine
    config.vm.box_check_update = false

    # La ligne suivante permet de désactiver le montage d'un dossier partagé (ne marche pas tout le temps directement suivant vos OS, versions d'OS, etc.)
    config.vm.synced_folder ".", "/vagrant", disabled: true

    config.vm.define "node1" do |node1|
        node1.vm.hostname = "node1.tp2.b2"
        node1.vm.network "private_network", ip: "192.168.2.21", netmask:"255.255.255.0"
        node1.vm.provider "virtualbox" do |vb|
          vb.memory = "1024"
        end
    end

    config.vm.define "node2" do |node2|
      node2.vm.hostname = "node2.tp2.b2"
      node2.vm.network "private_network", ip: "192.168.2.22", netmask:"255.255.255.0"
      node2.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
      end
    end
end
```
