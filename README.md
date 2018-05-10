sudo apt update 
sudo apt install openssh-server 


nano /etc/hosts 
192.168.1.73    nod01
192.168.1.74    nod02

# remova qualquer entrada do ssh 
# Gere as senhas 
ssh-keygen -t rsa 


ssh-copy-id nod02
# Logue-se no 
ssh nod02




##########  No servidor 
sudo apt-get install nfs-kernel-server

# Permita acesso sem senha 
eval `ssh-agent`
ssh-add .ssh/id_rsa


# criar uma pasta para compartilhamento 
mkdir /home/ifmt/mpiexemplo


#Adicionar uma entrada no arquivo /etc/exports
sudo nano /etc/exports

# Adicionar o conteúdo 
/home/ifmt/mpiexemplo  *(rw,sync,no_root_squash,no_subtree_check)

# Restarta o servidor 
sudo service nfs-kernel-server restart


#### Configuração do nfs dos clientes

ssh nodxxx
sudo apt-get install nfs-common

# Cria a pasta para efetuar a montagem
mkdir /home/ifmt/mpiexemplo

# Permitir que a pasta seja montada automaticamente 
sudo nano /etc/fstab
nod01:/home/ifmt/mpiexemplo /home/ifmt/mpiexemplo nfs

# monta o diretório 
mount -a 

sudo apt install libmpich-dev build-essential

# Crie um Arquivo para testar o cluster
nano hello.c 

#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    // Initialize the MPI environment
    MPI_Init(NULL, NULL);

    // Get the number of processes
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    // Get the rank of the process
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    // Get the name of the processor
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int name_len;
    MPI_Get_processor_name(processor_name, &name_len);

    // Print off a hello world message
    printf("Hello world from processor %s, rank %d"
           " out of %d processors\n",
           processor_name, world_rank, world_size);

    // Finalize the MPI environment.
    MPI_Finalize();
}


# pressione CTRL+X 
mpicc hello.c 

mpirun -np 4 --hosts nod01:4,nod02:2 ./a.out
