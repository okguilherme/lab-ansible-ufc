Com certeza, Guilherme. Vou manter exatamente a lógica que validamos e que funcionou nos seus testes de terminal, sem as adições automáticas de backup para não alterar o comportamento do seu Playbook original.

Aqui está a versão final e limpa do seu **README.md** para copiar e colar:

---

# 🚀 Automação de Certificados SSL - Infraestrutura UFC Quixadá

Este projeto automatiza a atualização de certificados SSL em **11 servidores** do campus da UFC em Quixadá. Ele utiliza **Ansible** para garantir que os certificados sejam entregues nos diretórios corretos e os serviços sejam reiniciados automaticamente, eliminando erros manuais e reduzindo o tempo de manutenção.

## 📋 Pré-requisitos na Máquina de Controle

Antes de começar, instale as dependências necessárias no computador que irá rodar o script:

```bash
sudo apt update
sudo apt install ansible sshpass -y
```

## 🔑 Configuração de Acesso (SSH RSA)

Para que o Ansible gerencie os servidores sem solicitar senha a cada tarefa, configure uma chave RSA na máquina do administrador:

1.  **Gerar a chave:** `ssh-keygen -t rsa -b 4096` (aperte Enter em todas as etapas).
2.  **Distribuir a chave:** Para cada servidor, envie a chave pública usando o usuário definido no `hosts.ini`:
    ```bash
    ssh-copy-id USER_DO_INVENTARIO@IP_DO_SERVIDOR
    ```
   

---

## 📂 Estrutura do Projeto

*   **`producao_estagio.yml`**: Playbook principal com a lógica de distribuição e suporte a Bundles.
*   **`hosts.ini`**: Arquivo de inventário organizado por grupos (Apache, Nginx, Docker) e usuários específicos.
*   **Arquivos de Certificado (Substituir pelos reais):**
    *   `certificado_estagio.cert` (Certificado do domínio)
    *   `intemediate_estagio.pem` (Certificado Intermediário)
    *   `gs_root_estagio.pem` (Certificado Raiz)
    *   `quixada_estagio.key` (Chave Privada)

---

## 🛠️ Como Executar

O projeto é **multi-usuário**. O Ansible lerá automaticamente o usuário de cada servidor configurado no arquivo `hosts.ini`.

### 1. Teste de Conectividade
Verifique se todos os servidores estão acessíveis e se a chave RSA está funcionando:
```bash
ansible servidores -i hosts.ini -m ping
```


### 2. Simulação de Segurança (Dry Run)
**Altamente recomendado.** Simula a entrega sem salvar nada e mostra as diferenças no texto:
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -K --check --diff
```


### 3. Execução Real
Aplica os certificados e reinicia os serviços necessários:
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -K
```


---

## ⚠️ Observações Técnicas Importantes

*   **Gestão de Usuários:** No arquivo `hosts.ini`, utilize o parâmetro `ansible_user=USER_AQUI` para definir usuários diferentes para cada servidor conforme a necessidade.
*   **Independência (HAProxy):** Este servidor requer um bundle unificado contendo a **Chave Privada** no final do arquivo `.pem`. A automação já trata isso via condicional `include_key: true`.
*   **Marizão (GitLab):** O certificado é enviado como um bundle público (sem a chave privada).
*   **Handlers:** Os serviços só serão reiniciados se o conteúdo do certificado for efetivamente alterado no destino.

---
**Data:** 23/04/2026
``
