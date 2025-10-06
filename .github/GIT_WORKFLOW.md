# Git Flow e Políticas de Merge

Este documento define o workflow Git oficial do projeto **Caronte** e as políticas de merge adotadas.

## 🌿 Estrutura de Branches

### Branches Principais

#### `master`

- **Propósito**: Branch de produção estável
- **Proteção**: Protegida, apenas merge via PR
- **Deploy**: Releases automáticos via tags
- **Histórico**: Deve manter histórico linear e limpo

#### `dev`

- **Propósito**: Branch de desenvolvimento ativo
- **Integração**: Todas as features são integradas aqui primeiro
- **Testes**: CI/CD completo antes de merge para `master`
- **Estabilidade**: Pode conter código em desenvolvimento

### Branches Temporárias

#### Feature Branches (`feature/`)

```bash
# Convenção de nomenclatura
feature/nome-da-funcionalidade
feature/aws-s3-integration
feature/azure-blob-storage
feature/add-retry-mechanism
```

- **Origem**: Criadas a partir de `dev`
- **Merge**: Via Pull Request para `dev`
- **Limpeza**: Deletadas após merge

#### Bug Fix Branches (`fix/`)

```bash
# Convenção de nomenclatura
fix/nome-do-bug
fix/aws-credentials-issue
fix/memory-leak-gcp-client
fix/timeout-configuration
```

- **Origem**: Criadas a partir de `dev` ou `master`
- **Merge**: Via Pull Request para `dev`
- **Limpeza**: Deletadas após merge

#### Hotfix Branches (`hotfix/`)

```bash
# Convenção de nomenclatura
hotfix/versao-nome-do-fix
hotfix/4.0.1-security-patch
hotfix/4.0.2-critical-aws-fix
```

- **Origem**: Criadas a partir de `master`
- **Merge**: Via Pull Request para `master` E `dev`
- **Deploy**: Release imediato após merge
- **Criticidade**: Apenas para correções críticas de produção

#### Release Branches (`release/`)

```bash
# Convenção de nomenclatura
release/versao
release/4.1.0
release/4.2.0-beta
```

- **Origem**: Criadas a partir de `dev`
- **Propósito**: Preparação para release (bump version, changelog)
- **Merge**: Para `master` e back-merge para `dev`
- **Limpeza**: Deletadas após merge

## 🔄 Workflow de Desenvolvimento

### 1. Feature Development

```bash
# 1. Atualizar branch dev local
git checkout dev
git pull origin dev

# 2. Criar branch feature
git checkout -b feature/nova-funcionalidade

# 3. Desenvolver com commits convencionais
git add .
git commit -m "feat: adiciona nova funcionalidade X"

# 4. Push da branch
git push origin feature/nova-funcionalidade

# 5. Criar Pull Request para dev
# Via interface do GitHub
```

### 2. Bug Fix Workflow

```bash
# 1. Atualizar branch dev local
git checkout dev
git pull origin dev

# 2. Criar branch fix
git checkout -b fix/corrige-bug-y

# 3. Implementar correção
git add .
git commit -m "fix: corrige problema Y no módulo Z"

# 4. Push e PR para dev
git push origin fix/corrige-bug-y
```

### 3. Hotfix Workflow

```bash
# 1. Criar hotfix a partir de master
git checkout master
git pull origin master
git checkout -b hotfix/4.0.1-security-fix

# 2. Implementar correção crítica
git add .
git commit -m "fix: corrige vulnerabilidade crítica de segurança"

# 3. PR para master (revisão rápida)
git push origin hotfix/4.0.1-security-fix

# 4. Após merge em master, back-merge para dev
git checkout dev
git pull origin master
git push origin dev
```

### 4. Release Workflow

```bash
# 1. Criar branch release a partir de dev
git checkout dev
git pull origin dev
git checkout -b release/4.1.0

# 2. Preparar release (bump version, changelog)
# Editar pyproject.toml, CHANGELOG.md
git add .
git commit -m "chore: prepare release 4.1.0"

# 3. PR para master
git push origin release/4.1.0

# 4. Após merge, criar tag
git checkout master
git pull origin master
git tag -a v4.1.0 -m "Release 4.1.0"
git push origin v4.1.0

# 5. Back-merge para dev
git checkout dev
git pull origin master
git push origin dev
```

## 📋 Políticas de Merge

### Branch `master`

**Política**: **Squash and Merge**

- ✅ **Vantagens**: Histórico limpo, um commit por feature
- ✅ **Aplicação**: PRs de `release/` e `hotfix/`
- ✅ **Revisão**: Mínimo 1 approver obrigatório
- ✅ **CI/CD**: Todos os checks devem passar

### Branch `dev`

**Política**: **Merge Commit**

- ✅ **Vantagens**: Preserva contexto da branch
- ✅ **Aplicação**: PRs de `feature/` e `fix/`
- ✅ **Revisão**: 1 approver obrigatório
- ✅ **CI/CD**: Todos os checks devem passar

### Proteções de Branch

#### `master`

```yaml
# Configurações no GitHub
- Require pull request reviews: true
- Required approving reviews: 1
- Dismiss stale reviews: true
- Require status checks: true
- Required status checks:
    - test
    - lint-and-type-check
    - security
    - build
- Require branches to be up to date: true
- Restrict pushes: true
- Allow force pushes: false
- Allow deletions: false
```

#### `dev`

```yaml
# Configurações no GitHub
- Require pull request reviews: true
- Required approving reviews: 1
- Require status checks: true
- Required status checks:
    - test
    - lint-and-type-check
    - security
- Require branches to be up to date: true
- Allow force pushes: false
```

## 📝 Padrões de Commit

### Conventional Commits

Seguimos o padrão [Conventional Commits](https://www.conventionalcommits.org/):

```bash
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### Types Permitidos

- `feat`: Nova funcionalidade
- `fix`: Correção de bug
- `docs`: Mudanças apenas na documentação
- `style`: Formatação, sem mudanças funcionais
- `refactor`: Refatoração sem mudanças funcionais
- `perf`: Melhoria de performance
- `test`: Adição/correção de testes
- `chore`: Mudanças no build, dependências, etc.
- `ci`: Mudanças no CI/CD
- `revert`: Reverter commit anterior

#### Scopes Sugeridos

- `aws`: Relacionado ao provedor AWS
- `azure`: Relacionado ao provedor Azure
- `gcp`: Relacionado ao provedor GCP
- `storage`: Funcionalidades de storage
- `functions`: Funcionalidades de functions
- `queue`: Funcionalidades de queue/messaging
- `auth`: Autenticação e autorização
- `config`: Configurações
- `deps`: Dependências

#### Exemplos

```bash
feat(aws): adiciona suporte ao S3 Glacier
fix(gcp): corrige timeout no Cloud Storage
docs: atualiza guia de instalação
test(azure): adiciona testes para Blob Storage
chore(deps): atualiza boto3 para v1.29.0
```

## 🚦 Status Checks Obrigatórios

### Para todas as branches protegidas:

1. **Tests** (`test`)

   - Testes unitários em Python 3.10, 3.11, 3.12
   - Cobertura mínima de 80%

2. **Lint & Type Check** (`lint-and-type-check`)

   - Ruff linting
   - Black formatting check
   - MyPy type checking

3. **Security** (`security`)

   - Bandit security scan
   - Safety dependency check

4. **Build** (`build`) - apenas para `master`
   - Poetry build
   - Package validation

## 🏷️ Tags e Releases

### Versionamento

Seguimos [Semantic Versioning](https://semver.org/):

- `MAJOR.MINOR.PATCH`
- `4.0.0a2` (alpha), `4.0.0b1` (beta), `4.0.0rc1` (release candidate)

### Tags

```bash
# Tags de release
v4.0.0, v4.1.0, v4.1.1

# Tags pre-release
v4.1.0-alpha.1, v4.1.0-beta.1, v4.1.0-rc.1
```

### GitHub Releases

- **Automático**: Via GitHub Actions ao criar tag
- **Changelog**: Gerado automaticamente baseado em commits
- **Assets**: Wheel e source distribution

## ❌ Práticas Proibidas

- ⛔ **Direct push** para `master` ou `dev`
- ⛔ **Force push** em branches protegidas
- ⛔ **Merge sem PR** em branches protegidas
- ⛔ **Commits sem conventional format**
- ⛔ **PRs sem testes** para features
- ⛔ **Bypass de status checks**
- ⛔ **Merge de PRs com conflicts**

## 🔧 Configuração Local

### Git Hooks (Pre-commit)

```bash
# Instalar pre-commit hooks
make dev-install

# Executar manualmente
make pre-commit
```

### Configuração Git Local

```bash
# Configurar template de commit
git config --local commit.template .gitmessage

# Configurar rebase automático
git config --local pull.rebase true

# Configurar prune automático
git config --local fetch.prune true
```

---

**📚 Referências:**

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Semantic Versioning](https://semver.org/)
