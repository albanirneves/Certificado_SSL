# Certificado_SSL
Guia de instalação de certificado para webserver

---
### Criar Chave
* Certifique-se que tenha o OpenSSL instalado em C:/OpenSSL ([Win32](https://slproweb.com/download/Win32OpenSSL_Light-1_1_0g.exe), [Win64](https://slproweb.com/download/Win64OpenSSL_Light-1_1_0g.exe), [Others](https://slproweb.com/products/Win32OpenSSL.html)).
* Abra o programa XCA Certificate and Key Management: [Download](https://sourceforge.net/projects/xca/).
* Crie um Banco de Dados em C:/OpenSSL/bin e coloque uma senha
* Crie uma nova chave com o nome api.dominio.com.br em Private Keys > New Key
* Exporte a chave para C:/OpenSSL/bin/api.dominio.com.br.pem
* Deixe o programa XCA aberto.
---

### Criar CSR
* Execute como Admin:
```
cd /OpenSSL/bin
    set OPENSSL_CONF=C:/OpenSSL/bin/openssl.cfg
    openssl req -new -key api.dominio.com.br.pem -out api.dominio.com.br.csr
```
* Insira as informações do certificado:
```
    Country: BR
    State: São Paulo
    Locality: São Paulo
    Organization Name: api.dominio.com.br
    Organization Unit Name: api.dominio.com.br
    Commom Name: api.dominio.com.br
    Email Adress: contato@dominio.com.br
    Password: ******
    Optional Company Name: api.dominio.com.br
```
* Verifique se foi criado o arquivo api.dominio.com.br.csr que será usado posteriormente
---

### Verificar Domínio
* Acesse [SSL For Free](https://www.sslforfree.com/)
* Digite o domínio (api.dominio.com.br) e clicar em "Create Free SSL Certificate"
* Escolha "Manual Verification" e "Manually Verify Domain"
* Baixe o arquivo de verificação e salvar na porta 80 em http://api.dominio.com.br/.well-known/acme-challenge/nomedoarquivo
(pode-se usar apache, nginx, etc)
* Verifique se a porta 80 está liberada no Firewall.
* Acesse o endereço de verificação: http://api.dominio.com.br/.well-known/acme-challenge/nomedoarquivo e veja se aparece o arquivo de verificação.

### Criar Certificados (Próprio e Raiz)
* Marque a opção "I have my own CSR" e confirme.
* Abra o arquivo C:/OpenSSL/bin/api.dominio.com.br.csr e cole todo seu conteúdo no campo "Enter your CSR here"
* Clique em "Download SSL Certificate"
* Salve o conteúdo de "Certificate" como C:/OpenSSL/bin/api.dominio.com.br.crt (Este é o certificado próprio)
* Salve o conteúdo de "CA Bundle"   como C:/OpenSSL/bin/ca_bundle.crt (Este é o certificado raiz)
* Volte ao programa XCA Certificate and Key Management
* Vá em Certificates > Import e selecione o arquivo C:/OpenSSL/bin/ca_bundle.crt
* Dê um clique no arquivo importado para selecioná-lo.
* Clique em Import e selecione o arquivo C:/OpenSSL/bin/api.dominio.com.br.crt
* Clique na setinha do certificado raiz e selecione o certificado próprio api.dominio.com.br
* Clique em Export
* Salve em C:/OpenSSL/bin/api.dominio.com.br.p12 (Export Format \*.p12), entrando com a senha
* O arquivo api.dominio.com.br.p12 é o certificado com raiz confiável que vamos instalar
* Salve-o no servidor em C:/ssl
* Caso seja a primeira vez que irá instalar o certificado para este site no servidor pule a próxima sessão.

### Desinstalação de Certificados Antigos
* Execute como Administrador:
```
    netsh http delete sslcert ipport=0.0.0.0:443
```
* Remova o certificado próprio e o raiz antigo através da impressão digital que pode ser encontrada em Painel de Controle > Opções da Internet > Conteúdo > Certificados > Let's Encrypt > Exibir > Detalhes > Impressão Digital
```
    certutil -delstore "My"   "D5 0E 9C 47 A4 E4 5D C4 07 BA 4F 03 C5 19 0E D4 58 08 D5 80"
    certutil -delstore "Root" "E6 A3 B4 5B 06 2D 50 9B 33 82 28 2D 19 6E FE 97 D5 95 6C CB"
```
* Remova os listeners da porta 443:
```
    netsh http delete urlacl http://+:443/service/
    netsh http delete urlacl https://+:443/service/
```
* Acesse novamente Painel de Controle > Opções da Internet > Conteúdo > Certificados
* Procure em todas as abas se ainda existe algum certificado Let's Encrypt instalado
* Caso haja remova-o

### Instalação do Certificado
* Importe o certificado como Admin (onde está ****** coloque a senha):
```
    cd /ssl
    certutil -f -p ****** -importpfx "%~dp0api.dominio.com.br.p12"
```
* Registre a impressão digital.
```
    cd /ssl
    netsh http add sslcert ipport=0.0.0.0:443 certhash=D5831A739821CEAFADA8DC0C3E61B44C1583AF49 appid={AA4AC37D-B812-46A7-BEFB-A68167A05BA7}
```
    Onde:
        "certhash" -> é o valor da impressão digital encontrada no certificado em Detalhes > Impressão Digital
        "appid" -> é a guid registrada no webserver
