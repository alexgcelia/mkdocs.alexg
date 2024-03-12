# mkdocs.alexg
Este repositorio es para la práctica mkdocs del módulo IAW

## MkDocs
#### Es un generador de sitios web estáticos que nos permite crear de forma sencilla un sitio web para documentar un proyecto. El contenido del sitio web está escrito en texto plano en formato Markdown y se configura con un único archivo de configuración en formato YAML.

### Temas para MkDocs
#### Se incluyen por defecto aqui: https://www.mkdocs.org/user-guide/styling-your-docs/#mkdocs.
En esta práctica se usa el tema Material, en esta página (https://squidfunk.github.io/mkdocs-material/getting-started/) encontraremos.

### Creamos el archivo de configuración mkdocs.yml
#### Configuración de entradas y páginas
```
site_name: IAW

nav:
    - Principal: index.md
    - Practica1.7: practica1-7.md
    - Practica1.9: practica1-9.md
    - Acerca de: about.md
```

#### Configuración de tema
```
theme:
  name: material
  language: es
```

#### Personalización del tema Material
```
  extra:
    social:
      - icon: assets/insta.jpg
        link: https://www.instagram.com/aalexx_63/
  features: 
    - navigation.indexes
    - navigation.instant
    - navigation.sections
    - navigation.tabs
    - announce.dismiss
  logo: assets/logo.png
  palette:
    - primary: deep orange
    - scheme: slate
  plugins:
  - search
  font:
    code: Oswald
```

### Como personalizar el tema Material
- Cambiar el color: (https://squidfunk.github.io/mkdocs-material/setup/changing-the-colors/).
- Cambiar la fuente de letra: (https://squidfunk.github.io/mkdocs-material/setup/changing-the-fonts/).
- Cambiar el idioma: (https://squidfunk.github.io/mkdocs-material/setup/changing-the-language/).
- Cambiar el logo y los iconos: (https://squidfunk.github.io/mkdocs-material/setup/changing-the-logo-and-icons/).
- Configurar la navegación: (https://squidfunk.github.io/mkdocs-material/setup/setting-up-navigation/).
- Configurar las búsquedas: (https://squidfunk.github.io/mkdocs-material/setup/setting-up-site-search/).
- Configurar Google Analytics: (https://squidfunk.github.io/mkdocs-material/setup/setting-up-site-analytics/).
- Configurar el versionado de la documentación: (https://squidfunk.github.io/mkdocs-material/setup/setting-up-versioning/).
- Configurar la cabecera: (https://squidfunk.github.io/mkdocs-material/setup/setting-up-the-header/).
- Configurar el pie de página: (https://squidfunk.github.io/mkdocs-material/setup/setting-up-the-footer/).
- Añadir un repositorio de git: (https://squidfunk.github.io/mkdocs-material/setup/adding-a-git-repository/).
- Añadir Disqus: (https://squidfunk.github.io/mkdocs-material/setup/adding-a-comment-system/).

### Generar la documentación (Comando: build)
```
docker run --rm -it -v "$PWD":/docs squidfunk/mkdocs-material build
```
### Publicar la documentación en GitHub Pages (Comando: gh-deploy)
```
docker run --rm -it -v ~/.ssh:/root/.ssh -v "$PWD":/docs squidfunk/mkdocs-material gh-deploy
```
### Creación de un workflow de CI/CD en GitHub Actions para pubicar un sitio web en GitHub Pages
#### Para crear un workflow en GitHub Actions podemos hacerlo de dos formas:
1. Desde la sección Actions -> New workflow.
2. Creando un archivo YAML dentro del directorio .github/workflows, en nuestro repositorio.
En nuestro caso, vamos a crear un archivo con el nombre build-push-mkdocs.yaml. Por lo tanto, nuestro repositorio quedará así:
```
name: build-push-mkdocs

# Eventos que desescandenan el workflow
on:
  push:
    branches: ["main"]

  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:

  # Job para crear la documentación de mkdocs
  build:
    # Indicamos que este job se ejecutará en una máquina virtual con la última versión de ubuntu
    runs-on: ubuntu-latest
    
    # Definimos los pasos de este job
    steps:
      - name: Clone repository
        uses: actions/checkout@v4

      - name: Configure Git Credentials
        run: |
          git config user.name github-actions[bot]
          git config user.email github-actions[bot]@fake.email.org
      - name: Install Python3
        uses: actions/setup-python@v4
        with:
          python-version: 3.x

      - name: Install Mkdocs
        run: |
          pip install mkdocs
          pip install mkdocs-material 
      - name: Build MkDocs
        run: |
          mkdocs build
      - name: Push the documentation in a branch
        uses: s0/git-publish-subdir-action@develop
        env:
          REPO: self
          BRANCH: gh-pages # The branch name where you want to push the assets
          FOLDER: site # The directory where your assets are generated
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub will automatically add this - you don't need to bother getting a token
          MESSAGE: "Build: ({sha}) {msg}" # The commit message
```
