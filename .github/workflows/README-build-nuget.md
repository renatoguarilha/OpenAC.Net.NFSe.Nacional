# 📦 Build NuGet Workflow - Guia de Configuração

Este documento descreve como configurar e usar o workflow `build-nuget.yml` para construir e publicar pacotes NuGet com a organização CMSol.Fiscal usando Trusted Publishing.

## 🎯 Visão Geral

O workflow permite:
- ✅ Build automático no push para `main` (versão com sufixo `-fork`)
- ✅ Build manual com sufixo customizável
- ✅ Build automático em tags `v*` (versão sem sufixo)
- ✅ Publicação opcional no NuGet.org via Trusted Publishing (OIDC)

## 🔐 Configuração Necessária

### 1. Trusted Publishing no NuGet.org

O Trusted Publishing deve estar configurado no NuGet.org com as seguintes informações:

```
Package Owner: CMSol.Fiscal (ou o dono do pacote)
Repository Owner: renatoguarilha
Repository: OpenAC.Net.NFSe.Nacional
Workflow File: build-nuget.yml
Environment: (opcional)
```

**Como configurar:**
1. Acesse [nuget.org](https://www.nuget.org)
2. Faça login com sua conta
3. Vá em "Trusted Publishing" no menu do usuário
4. Clique em "Add" ou "Create Policy"
5. Preencha os campos acima
6. Salve a configuração

> **Nota:** Segundo o issue, esta configuração já foi feita pelo usuário.

### 2. Secret do GitHub

Configure o seguinte secret no repositório GitHub:

**Nome do Secret:** `NUGET_USERNAME`  
**Valor:** Nome de usuário ou organização no NuGet.org

**Exemplos de valores:**
- Se o pacote pertence à organização: `CMSol.Fiscal`
- Se o pacote pertence ao usuário: `renatoguarilha`

**Como configurar:**
1. Acesse: `Settings` → `Secrets and variables` → `Actions`
2. Clique em "New repository secret"
3. Nome: `NUGET_USERNAME`
4. Valor: `CMSol.Fiscal` (ou o nome correto do dono do pacote)
5. Clique em "Add secret"

## 🚀 Como Usar

### Build Automático (sem publicação)

**Trigger:** Push para branch `main`

```bash
git push origin main
```

- ✅ Build automático é executado
- ✅ Versão: `{versão-base}-fork` (ex: `1.4.1-fork`)
- ✅ Pacote disponível como artifact por 90 dias
- ❌ NÃO publica no NuGet.org

### Build Manual (sem publicação)

1. Vá em: `Actions` → `📦 Build NuGet Package` → `Run workflow`
2. Escolha a branch: `main`
3. (Opcional) Altere o sufixo da versão
4. **Deixe o checkbox "Publicar no NuGet.org?" DESMARCADO**
5. Clique em "Run workflow"

- ✅ Build manual é executado
- ✅ Versão: `{versão-base}-{sufixo}` (ex: `1.4.1-beta`)
- ✅ Pacote disponível como artifact por 90 dias
- ❌ NÃO publica no NuGet.org

### Build e Publicação no NuGet.org

1. Vá em: `Actions` → `📦 Build NuGet Package` → `Run workflow`
2. Escolha a branch: `main`
3. (Opcional) Altere o sufixo da versão
4. **MARQUE o checkbox "Publicar no NuGet.org?"**
5. Clique em "Run workflow"

- ✅ Build manual é executado
- ✅ Versão: `{versão-base}-{sufixo}` (ex: `1.4.1-fork`)
- ✅ Pacote disponível como artifact por 90 dias
- ✅ **Publica automaticamente no NuGet.org**

### Build com Tag (Release)

```bash
git tag v1.4.2
git push origin v1.4.2
```

- ✅ Build automático é executado
- ✅ Versão: `1.4.2` (sem sufixo, direto da tag)
- ✅ Pacote disponível como artifact por 90 dias
- ❌ NÃO publica automaticamente (precisa de execução manual)

## 📦 Metadados do Pacote

O workflow sobrescreve os seguintes metadados do `.csproj`:

| Campo | Valor |
|-------|-------|
| **PackageId** | `CMSol.Fiscal.OpenAC.Net.NFSe.Nacional` |
| **Company** | `CMSol.Fiscal` |
| **Authors** | `Renato Guarilha` |
| **PackageTags** | `OpenAC.Net NFSe Nacional OpenNFSe CMSol Fiscal` |

Outros campos (descrição, licença, etc.) são mantidos do `.csproj` original.

## 🔒 Segurança

### Trusted Publishing (OIDC)

O workflow usa **Trusted Publishing via OIDC**, que oferece:

- ✅ **Sem API Keys de longa duração:** Não é necessário armazenar API keys no repositório
- ✅ **Chaves temporárias:** Cada publicação gera uma API key válida por ~1 hora
- ✅ **Autenticação forte:** Baseada em tokens OIDC assinados criptograficamente
- ✅ **Auditoria:** Todas as publicações são rastreáveis

### Permissões Mínimas

O workflow usa permissões mínimas:

```yaml
permissions:
  id-token: write  # Para obter token OIDC
  contents: read   # Para ler o código
```

### Controle de Publicação

- ✅ Publicação **NUNCA** acontece automaticamente
- ✅ Requer execução **manual** do workflow
- ✅ Requer **checkbox marcada** para publicar
- ✅ Falhas de publicação não afetam o build

## 📋 Inputs do Workflow

| Input | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| `version_suffix` | string | `fork` | Sufixo da versão (ex: `fork`, `beta`, `rc`) |
| `publish_to_nuget` | boolean | `false` | Marque para publicar no NuGet.org |

## 📊 Outputs e Summaries

### Build Job

Mostra:
- ✅ Detalhes do pacote (ID, versão, empresa, autor)
- ✅ Status de download (artifacts disponíveis por 90 dias)
- ✅ Status de publicação (se vai publicar ou não)

### Publish Job

Mostra:
- ✅ Confirmação de publicação bem-sucedida
- ✅ Link direto para o pacote no NuGet.org

## 🐛 Troubleshooting

### Erro: "NUGET_USERNAME secret not found"

**Causa:** Secret `NUGET_USERNAME` não está configurado.  
**Solução:** Configure o secret conforme instruções acima.

### Erro: "Unauthorized" ao publicar

**Causas possíveis:**
1. Trusted Publishing não configurado no NuGet.org
2. Configuração incorreta (repo, owner, workflow file)
3. Secret `NUGET_USERNAME` com valor incorreto

**Solução:** Verifique:
- Trusted Publishing está ativo no NuGet.org
- Valores batem exatamente (case-sensitive)
- Workflow file é `build-nuget.yml` (sem path)

### Erro: "Package already exists"

**Causa:** Versão do pacote já existe no NuGet.org.  
**Solução:** 
- Flag `--skip-duplicate` previne erro
- Se necessário, altere o sufixo da versão

### Job de publicação não executa

**Causa:** Condições não atendidas.  
**Requisitos:**
- ✅ Workflow executado via "Run workflow" (manual)
- ✅ Checkbox "Publicar no NuGet.org?" marcada

## 📚 Referências

- [NuGet Trusted Publishing](https://learn.microsoft.com/en-us/nuget/nuget-org/trusted-publishing)
- [GitHub OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [NuGet/login Action](https://github.com/marketplace/actions/nuget-login)
- [Microsoft Dev Blog - Trusted Publishing](https://devblogs.microsoft.com/dotnet/enhanced-security-is-here-with-the-new-trust-publishing-on-nuget-org/)

## 💡 Dicas

1. **Sempre teste primeiro sem publicar:** Execute o workflow sem marcar o checkbox para validar o build
2. **Use sufixos semânticos:** `beta`, `rc`, `fork` ajudam a identificar builds
3. **Versões de produção:** Use tags (`v*`) para releases oficiais
4. **Artifacts são temporários:** Baixe pacotes importantes antes de 90 dias
5. **--skip-duplicate é seguro:** Não há problema em executar múltiplas vezes

## 📞 Suporte

Para problemas ou dúvidas:
1. Verifique os logs do workflow no GitHub Actions
2. Consulte as referências acima
3. Abra uma issue no repositório
