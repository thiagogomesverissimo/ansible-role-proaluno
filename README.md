# Pró-Aluno FFLCH

Esse repositório contém a infraestrutura completa como
código do processo de build das 
máquinas das salas da pró-alunos da FFLCH.

Esta aberto a contribuições via de pull requests e issues do github. 
Foi inicialmente uma iniciativa dos alunos/as de graduação da FFLCH juntamente com a Seção  Técnica de Informática e apoio técnico do FLUSP (flusp.ime.usp.br) e do CCSL.

## Imagem

A imagem é construída com um arquivo preseed.cfg, versionado neste repositório
e disponível em:

 - http://public.fflch.usp.br/proaluno/preseed.cfg

## Provisionamento

Após a instalação da imagem, a ferramenta ansible é utilizada para configuração 
do resto do ambiente.

 - Integração com o servidor de autenticação samba da FFLCH
 - Instalação das impressoras
 - Instalação de pacotes por apt

https://youtu.be/abwQZz_8Xbc 

![](https://github.com/fflch/proaluno/raw/master/diagrama.png)

### Preparação do ambiente (testado com debian 10)

Instale na sua distro: ansible, vagrant

Instalação e configuração do libvirt para criar virtualização:

    $ sudo apt install virt-manager libvirt-dev
    $ sudo addgroup SEU-USER libvirt

Instalação do plugin do libvirt no ansible:

    $ vagrant plugin install vagrant-libvirt

Ligar as VMs:
    
    git clone git@github.com:VOCE-NO-GITHUB/proaluno.git
    cd proaluno
    vagrant up

Instalação das roles do ansible

    ansible-galaxy install -r requirements.yml

### Provisionando infraestrutura para desensolvimento:

Primeiramente suba a máquina do samba e rode a configuração:

    vagrant up samba
    ansible-playbook playbooks/dev/samba.yml

Neste momento temos configurado um servidor samba, para testar:

    vagrant ssh samba
    sudo samba-tool user list

Com o sistema https://github.com/uspdev/web-ldap-admin podemos administrar o servidor samba recém criado. Baixe o web-ldap-admin e configure como qualquer aplicação laravel. Configuração mínima para .env:

    LDAP_HOSTS=192.168.7.201
    LDAP_PORT=636
    LDAP_BASE_DN='DC=proaluno,DC=usp,DC=br'
    LDAP_USERNAME='CN=Administrator,CN=Users,DC=proaluno,DC=usp,DC=br'
    LDAP_PASSWORD='Pr0Aluno123'
    LDAP_USE_SSL=true
    LDAP_USE_TLS=false
    TIPO_NOMES_GRUPOS='siglas'
    REMOVE_ALL_GROUPS=1

Subindo o servidor do cups:

    vagrant up cups
    ansible-playbook playbooks/dev/cups.yml

Acesse a máquina do cups:

    vagrant ssh cups

Acessar cups pela interface: http://192.168.7.202:631/
Se pedir usuário e senha do cups digitar: root e Pr0Aluno123

Subir o sistema de quotas na máquina hospedeira, pois ele irá servir as requisições do quota check:

    php artisan serve --host 0.0.0.0

Subindo a máquina terminal que simula uma máquina física da proaluno:

    ansible-playbook playbooks/dev/terminal.yml

A senha de todos serviços nesse ambiente de teste é: Pr0Aluno123

Criando um usuário no samba com número USP 8001:



Na máquina hospederia em /etc/hosts:

    192.168.7.203 quotas.proaluno.usp.br
    192.168.7.203 faker.proaluno.usp.br

Rota de controle de quota /check/{user}/{printer}/{pages}:

    http://quotas.proaluno.usp.br/check/8001/impressora_pdf_tea4cups/2

Rota que verifica quantas páginas foram impressas hoje para o usuário 8001:

    http://quotas.proaluno.usp.br/pages/today/8001

Usuário admin usando no laravel:

    999 admin do laravel



