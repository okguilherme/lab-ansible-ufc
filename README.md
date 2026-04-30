
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

Para que o Ansible gerencie os servidores sem solicitar senha a cada tarefa, configure uma chave RSA:

1.  **Gerar a chave:** `ssh-keygen -t rsa -b 4096` (aperte Enter em todas as etapas).
2.  **Distribuir a chave:** Substitua `IP_DO_SERVIDOR` pelo IP de cada uma das 11 máquinas:
    ```bash
    ssh-copy-id osboxes@IP_DO_SERVIDOR
    ```

---

## 📂 Estrutura do Projeto

*   **`producao_estagio.yml`**: Playbook principal com a lógica de distribuição.
*   **`hosts.ini`**: Arquivo de inventário organizado por grupos (Apache, Nginx, Docker).
*   **Arquivos de Certificado (Substituir pelos reais):**
    *   `certificado_estagio.cert` (Certificado do domínio)
    *   `intemediate_estagio.pem` (Certificado Intermediário)
    *   `gs_root_estagio.pem` (Certificado Raiz)
    *   `quixada_estagio.key` (Chave Privada)

---

## 🛠️ Como Executar

Siga esta ordem para garantir uma implantação segura:

### 1. Teste de Conectividade
Verifique se todos os servidores estão acessíveis:
```bash
ansible servidores -i hosts.ini -m ping -u osboxes
```

### 2. Simulação (Dry Run)
**Altamente recomendado.** Este comando simula a entrega mas **não salva nada**. O parâmetro `--diff` permite visualizar as alterações nos arquivos (como a montagem dos Bundles no HAProxy e GitLab):
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -u osboxes -K --check --diff
```

### 3. Execução Real
Após validar a simulação, aplique os certificados:
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -u osboxes -K
```

---

## ⚠️ Observações Técnicas Importantes

*   **Independência (HAProxy):** Este servidor requer um bundle unificado contendo a **Chave Privada** no final do arquivo `.pem`. A automação já trata isso via condicional `include_key: true`.
*   **Marizão (GitLab):** O certificado é enviado como um bundle público (sem a chave privada).
*   **Handlers:** Os serviços (Apache, Nginx, Docker) só serão reiniciados se o conteúdo do certificado for efetivamente alterado.

---
**Desenvolvido por:** Guilherme Oliveira de Lima
**Data:** 23/04/2026
```
