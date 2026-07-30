# Estudo — Flashcards Anki (58 cartões)

*O baralho da preparação inteira, separado por tags de tópico — incluindo cartões de **digitar o comando** (o formato que você pediu, ex.: "a imagem é `fzi/rima-summer-school:latest` — digite o pull").*

**Baixar:** [flashcards_rima_go2.txt](anki/flashcards_rima_go2.txt)

## Importar no Anki (2 minutos)

1. Anki → **File → Import** → escolha o `.txt`.
2. Na janela de import: *Fields separated by* **Tab**; **Allow HTML in fields: ON** (essencial — os códigos vêm em HTML); Note type **Basic**; campo 3 mapeado para **Tags**.
3. Importar. Os cartões chegam com tags: `rede`, `docker`, `ros2`, `sdk`, `zenoh`, `nav2`, `pip`, `diad` — e a tag especial **`digitar`** nos de comando.

## Ativar o modo "digitar a resposta" (o recurso que você citou)

1. **Tools → Manage Note Types → Basic → clonar** como `Basic (digitar)`.
2. No clone, **Cards…** → no *Back Template*, troque `{{Back}}` por `{{type:Back}}`.
3. No navegador de cartões, filtre `tag:digitar`, selecione todos → **Notes → Change Note Type** → `Basic (digitar)`.

Dica honesta: o `type:` do Anki compara texto puro — brilha nos comandos de **uma linha** (a maioria dos `digitar`); nos cartões longos de código, o modo leitura normal funciona melhor. Por isso a separação por tag.

## Sugestão de uso até segunda

- Hoje/amanhã: `tag:diad` + `tag:zenoh` (o que você usa na chegada).
- No avião/trem: `tag:nav2` + `tag:docker`.
- Os demais entram no ritmo normal do Anki.
