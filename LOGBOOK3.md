# Trabalho realizado na Semana #3

## CTF - Semana 3

Sanity Check: A flag estava nas regras como descrito.

Desafio 1:
1. Acedemos ao url fornecido e explorámos os produtos vendidos
2. Encontrámos um produto com um comentário a denunciar problemas de segurança
3. Verificámos as versões do WordPress e Plugins do produto
4. Encontrámos no footer um aviso de que o próprio vendedor utiliza os mesmos plugins e WordPress
5. Procurámos CVE's correspondentes à informação recolhida
6. Encontrámos 2 possíveis candidatos e um deles estava correto

Desafio 2:
1. Utilizámos o Exploit fornecido em https://www.exploit-db.com/exploits/50299
2. Inicialmente tivemos problemas por tentar correr o ficheiro sem as permissões corretas e com a versão errada do Python
3. Corremos o exploit com o url do alvo e o id 1 (provável id do ADMIN)
4. Obtivémos 3 possíveis url's e um deles forneceu-nos privilégios de administrador
5. Acedemos à secção de posts fornecida http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php
6. Num post do admin, encontrámos a flag

## Identificação

- CVE-2021-34646 - WordPress Plugin WooCommerce Booster Plugin 5.4.3 - Authentication Bypass (https://www.exploit-db.com/exploits/50299)

- This vulnerability allows remote attackers to bypass authentication and act as other users.

- Affected systems:
    - WordPress Plugin WooCommerce Booster Plugin 5.4.3 (inclusive)

## Catalogação

- This exploit garnered an exceptionally high CVSS score of 9.8 out of 10, signifying its extreme criticality.

## Exploit

O exploit utilizado por nós foi encontrado no url https://www.exploit-db.com/exploits/50299


## Ataques


