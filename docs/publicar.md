# Publicar este site (GitHub Pages)

## 1. Rodar localmente (para conferir antes de publicar)

O Ubuntu 26.04 bloqueia `pip install` no Python do sistema — use um venv:

```bash
sudo apt install -y python3-venv git
python3 -m venv ~/.venvs/mkdocs
source ~/.venvs/mkdocs/bin/activate
pip install mkdocs-material
cd ~/FZI-Summer-School-Jorge-Brazilian          # a pasta deste projeto
mkdocs serve                 # abre em http://127.0.0.1:8000
```

## 2. Criar o repositório no GitHub

No site do GitHub: **New repository** → nome `FZI-Summer-School-Jorge-Brazilian` → público → **sem** README (o projeto já tem). Depois, na pasta do projeto:

```bash
git init
git add -A
git commit -m "Guia RimA Go2: etapas 0-3 validadas"
git branch -M main
git remote add origin git@github.com:jmetrimiranda/FZI-Summer-School-Jorge-Brazilian.git
git push -u origin main
```

(Se usa HTTPS em vez de SSH: `git remote add origin https://github.com/jmetrimiranda/FZI-Summer-School-Jorge-Brazilian.git`.)

Edite também no `mkdocs.yml` as duas linhas `site_url` e `repo_url`, trocando `jmetrimiranda`.

## 3. Publicar

**Caminho A — automático (recomendado):** o projeto já inclui `.github/workflows/deploy.yml`. Todo `git push` na `main` reconstrói o site sozinho. Só falta, **uma vez**, ativar: GitHub → Settings → Pages → *Build and deployment* → Source: **Deploy from a branch** → Branch: **gh-pages** / root → Save. (A branch `gh-pages` aparece após o primeiro push, criada pela Action.)

**Caminho B — manual:** com o venv ativo, `mkdocs gh-deploy --force` publica na hora. A configuração do Pages é a mesma.

O site fica em `https://jmetrimiranda.github.io/FZI-Summer-School-Jorge-Brazilian/`.

!!! success "✅ Teste de aceite"
    Abrir a URL acima no navegador e ver a página inicial com a tabela de progresso.

## 4. Fluxo de atualização (nosso combinado)

1. Fechamos uma etapa na prática (testes de aceite verdes).
2. Você recebe o `.md` novo/atualizado → substitui em `docs/`.
3. `git add -A && git commit -m "Etapa N validada" && git push` → site atualiza sozinho.
