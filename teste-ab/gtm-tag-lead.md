# GTM — tag de log do teste A/B (evento Lead)

Uma tag nova no contêiner `GTM-KFKMT6BZ`, pra marcar **qual página (v1–v4)** gerou cada lead. Não mexe na tag do pixel que já existe — só adiciona uma tag extra no mesmo gatilho.

## Passo a passo

1. **Tags → Nova**
2. **Configuração da tag → Personalizada → Imagem** ("Custom Image Tag")
3. **URL da imagem:**
   ```
   https://brito-midias-n8n.ertdxv.easypanel.host/webhook/fda-ja-ab-teste?event=lead&variante={{Page Path}}
   ```
   (`{{Page Path}}` é a variável embutida do GTM — não precisa criar nada novo. Se o contêiner já tiver variáveis de UTM da URL configuradas, pode anexar `&utm_source={{...}}` etc. do mesmo jeito que a tag do pixel usa, mas não é obrigatório — `event` + `variante` já bastam pra comparar as páginas.)
4. **Acionamento:** use **o mesmo gatilho da tag que já dispara o `Lead`** hoje (procure a tag existente do pixel/Stape para conferir qual é).
5. **Nome da tag:** `Log lead — Teste A/B (n8n)`
6. Salvar → **Enviar** (publicar o contêiner).

## Verificação

No **Preview mode** do GTM, abra uma das páginas (`.../v1/` a `.../v4/`), dispare o evento de lead de teste, e confira nos Tags que a tag `Log lead — Teste A/B (n8n)` disparou junto com a tag existente. A linha aparece na aba **"Teste A/B"** da planilha do projeto em poucos segundos.
