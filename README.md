
---

```markdown
# 🚀 Automação de Certificados SSL - Infraestrutura UFC Quixadá

Este projeto automatiza o ciclo de atualização e distribuição de certificados SSL em 11 servidores do campus da **UFC em Quixadá**. Utilizando **Ansible**, a solução garante que 
os certificados sejam entregues nos diretórios corretos e os serviços (Apache, Nginx, Docker, HAProxy) sejam reiniciados automaticamente, eliminando erros manuais e reduzindo drasticamente o tempo de manutenção.

---

## 📋 Pré-requisitos (Máquina de Controle)

Antes de executar o playbook, certifique-se de que a máquina administradora possui as dependências instaladas:

```bash
sudo apt update
sudo apt install ansible sshpass -y
```

## 🔑 Configuração de Acesso (SSH RSA)

Para que o Ansible gerencie os servidores de forma automatizada (sem solicitar senha a cada tarefa), é necessário configurar chaves RSA:

1.  **Gerar a chave:** ```bash
    ssh-keygen -t rsa -b 4096
    # Pressione Enter em todas as etapas para manter o padrão.
    ```
2.  **Distribuir a chave:** Para cada servidor listado no `hosts.ini`, envie a chave pública:
    ```bash
    ssh-copy-id USUARIO@IP_DO_SERVIDOR
    ```

---

## 📂 Estrutura do Projeto

* **`producao_estagio.yml`**: Playbook principal contendo a lógica de distribuição, tratamento de Bundles e reinicialização de serviços.
* **`hosts.ini`**: Arquivo de inventário organizado por grupos (Apache, Nginx, Docker) com variáveis de usuário específicas.
* **Arquivos de Certificado** (Devem ser substituídos pelos arquivos reais):
    * `certificado_estagio.cert`: Certificado do domínio.
    * `intemediate_estagio.pem`: Certificado Intermediário.
    * `gs_root_estagio.pem`: Certificado Raiz.
    * `quixada_estagio.key`: Chave Privada (Mantenha este arquivo seguro).

---

## 🛠️ Como Executar

O projeto é **multi-usuário**. O Ansible utilizará automaticamente o usuário definido para cada servidor dentro do arquivo `hosts.ini`.

### 1. Teste de Conectividade
Verifique se todos os servidores estão acessíveis via SSH:
```bash
ansible servidores -i hosts.ini -m ping
```

### 2. Simulação de Segurança (Dry Run)
Execução em modo de teste para validar mudanças sem aplicá-las:
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -K --check --diff
```

### 3. Execução Real
Aplica os certificados e reinicia os serviços:
```bash
ansible-playbook -i hosts.ini producao_estagio.yml -K
```

---

## ⚠️ Observações Técnicas Importantes

* **Gestão de Usuários:** No `hosts.ini`, utilize o parâmetro `ansible_user=NOME_DO_USUARIO` para personalizar o acesso por host.
* **Caso Especial (HAProxy):** Requer um bundle unificado com a chave privada ao final. A automação utiliza a condicional `include_key: true` para este caso.
* **GitLab (Marizão):** Configurado para receber o bundle público (sem a chave privada integrada).
* **Idempotência e Handlers:** Os serviços (Nginx/Apache/etc) **só serão reiniciados** se o conteúdo do certificado for alterado no destino, evitando downtime desnecessário.

---

**Desenvolvido por:** Guilherme Oliveira de Lima  
**Data:** 23/04/2026  
**Local:** UFC - Campus Quixadá
```
