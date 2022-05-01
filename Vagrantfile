# Parameters
CPU_CORES = 6
RAM_MB = "8192"

# Vagrant 
Vagrant.configure("2") do |config|
  config.vm.define "FreeSurfer" do |cfg|
    cfg.vm.box = "ubuntu/bionic64"
    cfg.vm.hostname = "FreeSurfer"
    cfg.vm.boot_timeout = 1200
    cfg.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.name = "FreeSurfer"
      vb.memory = RAM_MB
      vb.cpus = CPU_CORES
    end
    # Install Freesurfer once at creation  
    cfg.vm.provision "shell", inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      rm -rf /var/lib/apt/lists/*
      apt-get update
      apt-get -y upgrade
      apt-get -y autoremove
      apt-get clean
      apt-get -y install wget tar gzip tcsh libgomp1 binutils
      if [ ! -f "/vagrant/freesurfer-linux-ubuntu18_amd64-7.2.0.tar.gz" ]; then
        wget https://surfer.nmr.mgh.harvard.edu/pub/dist/freesurfer/7.2.0/freesurfer-linux-ubuntu18_amd64-7.2.0.tar.gz -P /vagrant
      fi
      tar -zxpf /vagrant/freesurfer-linux-ubuntu18_amd64-7.2.0.tar.gz -C /home/vagrant/
      if [ -f "/vagrant/license.txt" ]; then
        cp /vagrant/license.txt /home/vagrant/freesurfer/.license
      fi
    SHELL
    # Run script at every creation or reload to execute recon-all 
    cfg.vm.provision "shell", run: "always", args: CPU_CORES, inline: <<-SHELL
      # Variables
      CPU_CORES=$1
      echo "Number of cores to use: $CPU_CORES"
      # Cleanup
      cd /vagrant
      sudo rm -rf /vagrant/output/
      sudo mkdir /vagrant/output
      sudo rm -rf /home/vagrant/freesurfer/subjects/Subject1
      sudo rm -rf /tmp/command.sh
      # load FreeSurfer
      export FREESURFER_HOME=/home/vagrant/freesurfer
      source $FREESURFER_HOME/SetUpFreeSurfer.sh
      # run recon-all command
      cd /vagrant/input/
      FIRSTFILE=$(ls -1 | head -1)
      recon-all -subjid Subject1 -openmp $CPU_CORES -all -i $FIRSTFILE
      # generate STL file
      mris_convert --combinesurfs /home/vagrant/freesurfer/subjects/Subject1/surf/lh.pial /home/vagrant/freesurfer/subjects/Subject1/surf/rh.pial /home/vagrant/cortical-Subject1.stl
      # copy files
      cp -rL /home/vagrant/freesurfer/subjects/ /vagrant/output/freesurfer/
      cp /home/vagrant/cortical-Subject1.stl /vagrant/output/stl/brain.stl
    SHELL
  end
end
