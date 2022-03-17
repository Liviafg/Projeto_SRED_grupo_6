# SREDgrupo6
Desenvolvimento de rede de computador virtualizada.

# Configuração de Interface de Rede


<p><center> Tabela 1: Definições da rede interna grupo 6, 924</center></p>

| DESCRIÇÃO   | IP             |
|:------------|:---------------| 
| rede        | 10.9.24.0      |
| máscara     | 255.255.255.0  |
| Gateway     | 10.9.24.120    |
| Broadcast   | 10.9.24.255    |
| NameServer1 | 10.9.24.113    |
| NameServer2 | 10.9.24.121    |
| Samba       | 10.9.24.115    |
| BD          | 10.9.24.222    | 



Os nomes das máquinas ou dispositivos que serão configuradas no DNS deverão ser nomeados de acordo com o domínio. A Tabela 2 apresenta os nomes das máquinas para o domínio de exemplo.

<p><center> Tabela 2: Definições do domínio: <b>name.grupoX.turma.ifalarapiraca.local</b></center></p>

|      Apelido      |               NOME               |
|:------------------|:---------------------------------|
| gateway (gw)      | gw.grupo6.turma924.ifalarapiraca.local    |
| nameserver1 (ns1) | ns1.grupo6.turma924.ifalarapiraca.local   |
| nameserver2 (ns2) | ns2.grupo6.turma924.ifalarapiraca.local   |
| hostsamba   (hs1) | samba.grupo6.turma924.ifalarapiraca.local |

- [Clique Aqui Para ver a Planilha detalhada](https://docs.google.com/spreadsheets/d/14w_kkyJeZiC50_NOaJPD21q7aiylo4qtkw7SilrcD3U/edit?hl=pt-br#gid=0)

## Configuração estática do DNS na interface de rede. 

* Para que a máquina acesse os sites e hosts remotos por meio de nomes (Ex. www.google.com) é necessário adcionar os nameservers na configuração da interface de rede.
* Para isso é configure o arquivo YAML que encontra-se na pasta **/etc/netplan/**.
* Verifique o nome correto do arquivo no seu servidor. No exemplo a seguir, o nome do arquivo é ***00-installer-config.yaml***

-  Edite o arquivo  ***00-installer-config.yaml*** 

```bash
$ sudo nano /etc/netplan/00-installer-config.yaml
```

-  Adicione as linhas para a configuração estática do IP. [Baixe o arquivo 00-installer-config.yaml](https://github.com/alaelson/labredes2020/blob/master/network/interface-config/00-installer-config.yaml)
```
network:
  ethernets:
    ens160:
      dhcp4: false
      addresses: [10.9.24.113/24]
      gateway4: 10.9.24.1
      nameservers:
          addresses:
           - 10.9.24.113
           - 10.9.24.121
          search: [grupo6.turma924.ifalara.local]
    ens192:
          addresses: [192.168.24.46/29]
  version: 2

```
-  Após salvar o arquivo é necessário aplicar as configurações, com o **netplan apply**. Depois veja a configuração das interfaces com ****ifconfig -a***

```bash
$ sudo netplan apply
$ ifconfig -a
```
-  Adicione as linhas para a configuração estática do IP. [Baixe o arquivo 00-installer-config.yaml](https://github.com/alaelson/labredes2020/blob/master/network/interface-config/00-installer-config.yaml)

```
network:
  ethernets:
    ens160:
      dhcp4: false
      addresses: [10.9.24.113/24]
      gateway4: 10.9.24.1
      nameservers:
          addresses:
           - 10.9.24.113
           - 10.9.24.121
          search: [grupo6.turma924.ifalara.local]
    ens192:
          addresses: [192.168.24.46/29]
  version: 2

```
-  Após salvar o arquivo, aplique as novas configurações, com o **netplan apply**. Depois veja a configuração das interfaces com ****ifconfig -a***

```bash
$ sudo netplan apply
$ ifconfig -a


```bash
$ sudo systemctl enable bind9
```
## 4. Implementação dos Serivços de Rede (Cada serviço uma sessão)
   ## *4.2 Confligurando o DNS Master:*
   
### Instalação necessária do Bind9 
   * Vale ressaltar que o BIND9 é a aplicação de DNS que roda no servidor.
   * Para instalar o bind9 de forma fácil é via comando:
```bash
$ sudo apt-get install bind9 dnsutils bind9-doc 
```
   * Verifique o status do serviço:
```bash
$ sudo systemctl status bind9
```
   * Se não estiver rodando:
```bash
$ sudo systemctl enable bind9


## Diretórios do bind
   * Os arquivos do bind ficam na no diretório **/etc/bind**. 
```bash
$ ls /etc/bind
```
```
bind.keys  
db.0  
db.127  
db.255  
db.empty 
db.local
named.conf  
named.conf.default-zones 
named.conf.local 
named.conf.options 
rndc.key  
zones  
zones.rfc1918

```

### Zonas
   * As zonas são especificadas em arquivos **db**. Vamos criar um diretório para armazendar os arquivos de zonas, que sera o diretório ***/etc/bind/zones***  
```bash
$ sudo mkdir /etc/bind/zones
```

#### Criar arquivos db
   * Criar o arquivo **db** no diretório ***/etc/bind/zones***. 
   * Os arquivos **db** são bancos de dados de resolução de nomes, ou seja, quando se sabe o nome da máquina mas não se conhece o IP. Cada zona no DNS deve ter seu próprio arquivo **db**, por exemplo: a zona *meusite.com.br* terá o arquivo **db.meusite.com.br**, já a zona *outrosite.net* terá o arquivo **db.outrosite.net**. 
   * No nosso caso o domínio/zona local será grupo5.turma924.ifalara.local. Assim o arquivo db será db.labredes.ifalarapiraca.local
   
##### zona direta
   * o arquivo db.grupo6.turma924.ifalara.local conterá os nomes das máquinas do domínio grupo5.turma924.ifalara.local
   * Para isso faremos uma cópia do arquivo /etc/bind/db.empty
```bash
$ sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo5.turma924.ifalara.local 
```

##### zona reversa
   * Utilizado quando não se conhece o IP mas sabe-se o nome do host.
   * vamos criar a zona reversa a partir do arquivo /etc/bind/db.127
```bash
  $ sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.24.rev
```

   * Assim, o arquivo **db.10.9.24.rev** conterá a zona reversa da rede 10.9.24.0. 

   
### Editar arquivos db:

   #### zona direta: db.grupo6.turma924.ifalara.local
   * edite o arquivo  **db.grupo6.turma924.ifalara.local** para adcionar as informações do seu domínio
      
```bash   
    $ sudo nano db.grupo5.turma924.ifalara.local
```
---
```


;

;
; BIND data file for internal network
;
$ORIGIN grupo6.turma924.ifalara.local.
$TTL    3h
@       IN      SOA     ns1.grupo6.turma924.ifalara.local. root.grupo6.turma924>
                              2022031401                ; Serial
                              3h        ; Refresh
                              1h        ; Retry
                              1w        ; Expire
                              1h )      ; Negative Cache TTL
;nameservers
@       IN      NS      ns1.grupo6.turma924.ifalara.local.
@       IN      NS      ns2.grupo6.turma924.ifalara.local.
;hosts
ns1.grupo6.turma924.ifalara.local.        IN    A       10.9.24.113
ns2.grupo6.turma924.ifalara.local.        IN    A       10.9.24.121
gw.grupo6.turma924.ifalara.local.         IN    A       10.9.24.120
www.grupo6.turma924.ifalara.local.        IN    A       10.9.24.221
bd.grupo6.turma924.ifalara.local.         IN    A       10.9.24.222

```

---
   #### zona reversa: db.10.9.24.rev
   * edite o arquivo **db.10.9.24.rev** para adcionar as informações da zona reversa
   
---
```
;

; BIND reverse data file for local loopback interface
;
$TTL    60
@       IN      SOA     ns1.grupo6.turma924.ifalara.local. root.grupo6.turma924>
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.grupo6.turma924.ifalara.local.
@       IN      NS      ns2.grupo6.turma924.ifalara.local.

113     IN      PTR     ns1.grupo6.turma924.ifalara.local.
121     IN      PTR     ns2.grupo6.turma924.ifalara.local.
120     IN      PTR     gw.grupo6.turma924.ifalara.local.
115     IN      PTR     samba.grupo6.turma924.ifalara.local
222     IN      PTR     bd.grupo6.turma924.ifalara.local
221     IN      PTR     www.grupo6.turma924.ifalara.local


```
---

## Configuração do samba

1.  Após conectar-se ao OpenVPN, abra o programa PuTTY e insira o endereço IP da sua máquina virtual.
2.  Após abrir, entre com o usuário administrador e senha adminifal. 
```
  user: administrador
  password: adminifal
```
 



