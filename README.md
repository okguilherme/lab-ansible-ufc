---
# 🚀 Automação de Certificados SSL - Infraestrutura UFC Quixadá

Este projeto automatiza a atualização de certificados SSL em **11 servidores** do campus. A versão atual utiliza **Ansible Vault** para criptografia de credenciais e um inventário dinâmico, garantindo que senhas de `sudo` diferentes sejam gerenciadas de forma segura e automática.

## 📋 Pré-requisitos na Máquina de Controle

Instale as dependências no computador que irá rodar o script:
```bash
sudo apt update
sudo apt install ansible sshpass -y
```
## ⚠️ Observações Técnicas

*   **Gestão de Variáveis:** No arquivo `hosts.ini`, utilizamos `ansible_user` para o login e `ansible_become_password="{{ nome_da_variavel }}"` para o privilégio de root, buscando o valor dentro do Vault.
*   **Segurança:** Nunca suba o arquivo `vars_senhas.yml` para repositórios públicos sem que ele esteja criptografado.
*   **Handlers:** O `restart` dos serviços só ocorre se o arquivo de certificado for modificado, evitando downtime desnecessário.

## 🔑 Configuração de Segurança e Acesso

### 1. Acesso sem senha (SSH RSA)
Para o Ansible entrar nos servidores, gere e distribua sua chave pública:
*   **Gerar:** `ssh-keygen -t rsa -b 4096`
*   **Distribuir:** `ssh-copy-id USER@IP_DO_SERVIDOR` (Repita para cada IP do inventário)

### 2. Cofre de Senhas (Ansible Vault)
As senhas de `sudo` de cada servidor não estão mais em texto puro. Elas ficam no arquivo criptografado `vars_senhas.yml`.
*   **Senha Mestra do Arquivo:** `123`
*   **deve inserir as senhas reais:**
    Para editar o arquivo e colocar as senhas corretas de produção, use o comando:
    ```bash
    EDITOR=nano ansible-vault edit vars_senhas.yml
    ```
    *O editor **Nano** abrirá; basta substituir os valores, salvar (`Ctrl+O`, `Enter`) e sair (`Ctrl+X`).*

🔐 Como Alterar a Senha Mestra (Rekey)
Caso precise alterar a senha de proteção do arquivo vars_senhas.yml, utilize o comando rekey. Isso atualizará a criptografia sem que você precise abrir o arquivo.

No terminal, dentro da pasta do projeto, execute:

Bash
ansible-vault rekey vars_senhas.yml
Siga a sequência de segurança que aparecerá no terminal:

Vault password: Digite a senha atual (ex: 123).

New Vault password: Digite a sua nova senha.

Confirm New Vault password: Digite a nova senha novamente para confirmar.

[!TIP]
Após alterar a senha, o arquivo será modificado. Não esqueça de realizar um novo git add e git push para atualizar a versão criptografada no GitHub.

## 📂 Estrutura e Preparação dos Arquivos

**IMPORTANTE:** Os arquivos de certificado devem estar **na mesma pasta raiz do projeto** para que o Playbook os localize corretamente. Substitua os arquivos de teste pelos oficiais mantendo estes nomes:
*   `certificado_estagio.cert` (Certificado do domínio)
*   `intemediate_estagio.pem` (Intermediário)
*   `gs_root_estagio.pem` (Raiz)
*   `quixada_estagio.key` (Chave Privada)

## 🛠️ Como Executar

O projeto agora associa automaticamente o usuário e a senha de cada servidor através do `hosts.ini` e do `vars_senhas.yml`.

### 1. Teste de Conectividade
```bash
ansible servidores -i hosts.ini -m ping
```

### 2. Simulação (Dry Run)
Veja as mudanças que seriam feitas sem aplicar nada:
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -e "@vars_senhas.yml" --ask-vault-pass --check --diff
```

### 3. Execução Real
Aplica os certificados e reinicia os serviços (Apache, Nginx, Docker):
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -e "@vars_senhas.yml" --ask-vault-pass
```
*Ao rodar, o sistema pedirá a **Vault Password**. Digite `123`.*

---

**Data da última atualização:** 05/05/26
