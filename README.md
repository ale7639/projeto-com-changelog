name: Update README with Changelog

on:
  push:
    branches:
      - main  #  Mude para 'master' se for o caso do seu projeto

jobs:
  update-readme:
    runs-on: ubuntu-latest

    # Condição para NÃO rodar se o commit foi feito pelo bot
    if: "!contains(github.event.head_commit.message, '[skip ci]')"

    steps:
      # Passo 1: Clona o repositório para dentro da action
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Passo 2: Pega a mensagem do último commit
      - name: Get last commit message
        id: last_commit
        run: echo "message=$(git log -1 --pretty=%B)" >> $GITHUB_OUTPUT

      # Passo 3: Atualiza o arquivo README.md
      - name: Update README with changelog
        run: |
          COMMIT_MESSAGE="${{ steps.last_commit.outputs.message }}"

          if ! grep -q "## Changelog" README.md; then
            echo -e "\n## Changelog\n" >> README.md
          fi

          sed -i '/^## Changelog/a TEMP_MARKER' README.md
          sed -i "s|TEMP_MARKER|- $(date '+%Y-%m-%d %H:%M') - $COMMIT_MESSAGE|" README.md

      # Passo 4: Faz o commit e push da mudança
      - name: Commit and Push changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"

          if ! git diff --quiet README.md; then
            git add README.md
            git commit -m "docs: update changelog [skip ci]"
            git push
          else
            echo "No changes to commit."
          fi
