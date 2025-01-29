### **Objetivo do Laboratório**

Criar uma topologia de rede com:

1.  Um servidor **DNS** baseado no container **BIND9**.
2.  Um cliente que faz consultas DNS para o servidor.

----------

## **Passo a Passo para Configurar**

Temos um arquivo de topologia criado para o laboratório. Ao executá-lo o container lab irá criar dois containers
- DNS Server
- Cliente que irá usar o servidor DNS

### **Tarefa**

#### 1. Criar diretório **bind**

Crie o diretório `bind` no mesmo local do arquivo de topologia (na raiz do projeto que você baixou)
Dê permissão ao usuário docker
```bash
sudo chdown -R $USER:$USER bind/
```
#### 2. Iniciar o Containerlab

Execute o laboratório com o comando:

```bash
sudo containerlab deploy -t dns-lab.clab.yaml
```

O comando criará:

-   Um container para o servidor **BIND9** com a configuração DNS.
-   Um container **Ubuntu** como cliente para testes.

 e insira os seguintes arquivos de configuração dentro desta pasta bind:

#### 3. Configurar servidor DNS

Na pasta bind edite (caso já exista ) ou crie o arquivo `named.conf.local` 
**Arquivo `named.conf.local`**:

```bash
zone "nome_e_sobre_nome_do_aluno.com" {
    type master;
    file "/etc/bind/db.nome_e_sobre_nome_do_aluno.com";
};

```

Exemplo de como a arquivo deve ficar se fosse com o aluno fulano de tal

```bash
zone "fulano_detal.com" {
    type master;
    file "/etc/bind/db.fulano_detal.com";
};

```
Na pasta bind edite (caso já exista ) ou crie o arquivo db.nome_e_sobre_nome_do_aluno.com da como definido abaixo. 
**Arquivo `db.nome_e_sobre_nome_do_aluno.com`**:

```plaintext
$TTL    604800
@       IN      SOA     nome_e_sobre_nome_do_aluno.com. root.nome_e_sobre_nome_do_aluno.com. (
                        2         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL
;
@       IN      NS      ns.nome_e_sobre_nome_do_aluno.com.
ns      IN      A       192.168.1.**
www     IN      A       192.168.1.**

```

- Altere a linha `ns      IN      A       192.168.1.**` e e troque o ** pelo dois últimos números de sua matrícula 
- Altere a linha `www     IN      A       192.168.1.**` e troque o ** pelo dois últimos números de sua matrícula + 1 



#### 4. Configurar IPs nos Containers

Após a topologia estar ativa, entre no container **dns-server** e configure o IP:
Troque os `**` pelos dois últimos digitos de sua matrícula
```bash
docker exec -it clab-dns-lab-dns-server bash
ip addr add 192.168.1.**/24 dev eth1
ip link set eth1 up

```


Saia do container com o comando `exit`

Repita o processo no cliente, configurando o IP:

Troque os `**` pelos dois últimos digitos de sua matrícula + 1

```bash
docker exec -it clab-dns-lab-client bash
ip addr add 192.168.1.**3**/24 dev eth1
ip link set eth1 up

```

Ainda no cliente, configure o **DNS** para apontar para o servidor:

Aqui como parte da tarefa, você já deve saber qual é o IP_SERVID_DNS. Então rode o comando abaixo trocando essa variável pelo valor correto previamente configurado.

```bash
echo "nameserver IP_SERVIDOR_DNS" > /etc/resolv.conf

```

----------

#### 5. Testar o DNS

Ainda dentro do container do cliente, execute os seguintes comandos para testar:

1.  Verificar resolução de nomes:
    
    ```bash
    nslookup www.exemplo.com > /etc/bind/lookup.txt
    
    ```
    
    Saída esperada dentro do arquivo lookup.txt:
    
    ```
    Server:         192.168.1.2
    Address:        192.168.1.2#53
    
    Name:   www.nome_e_sobre_nome_do_aluno.com
    Address: 192.168.1.3
    
    ```
    
2.  Rode ping para o domínio:
    
    ```bash
    ping www.nome_e_sobre_nome_do_aluno.com > /etc/bind/ping.txt
    
    ```

#### 6. Limpar o Laboratório

Adicione ao repositório do lab os seguintes arquivos

```bash
git add ./bind/ping.txt
git add ./bind/lookup.txt
git add ./bind/db.nome_e_sobre_nome_do_aluno.com
git add ./bind/named.conf.local
```
Realize o commit da atividade

```bash
git commit -m "Finalizando Lab 01"
```

Envie para o repositório remoto onde o professor irá corrigir

```bash
git push origin main
```

#### 7. Limpar o Laboratório

Após finalizar o lab e fazer o commit, certifique-se que a tarefa foi realmente enviada e remova a topologia criada com o container lab usando o seguinte comando :

```bash
sudo containerlab destroy -t dns-lab.clab.yaml

```
