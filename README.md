# Arya Living — landing pages

Site estático. Duas páginas:

- `/` → **index.html** — inscrição na apresentação ao vivo
- `/tabela` → **tabela.html** — pedido de tabela (segmentável por `?p=investidor|exterior|veraneio|diversificar`)

## Deploy no Coolify (GitHub)

1. Crie um repositório no GitHub e suba **o conteúdo desta pasta** na raiz (não a pasta `deploy` em si).
2. No Coolify: **New Resource → Application → GitHub**, escolha o repositório e a branch.
3. **Build Pack: Dockerfile** (o Dockerfile está na raiz). Porta: **80**.
4. Aponte o domínio e faça o deploy. Pronto.

Toda alteração no repositório redeploya automaticamente.

## Editar data, hora, vagas, webhook etc. depois de publicado

Abra `https://seudominio.com/?admin=1`. Edite os campos e clique **Baixar config.json**.
Substitua o `config.json` do repositório pelo arquivo baixado e faça commit — o Coolify redeploya.

Alternativa sem commit: preencha **Webhook admin** com uma URL do n8n que receba o JSON
(`{ type: "arya-live-config", config: {...} }`) e grave `config.json` no repositório via API do GitHub
(ou num bucket público — nesse caso troque `configUrl` no HTML para a URL do bucket).

O `config.json` é servido com `no-store`: qualquer mudança vale no próximo carregamento.

## Webhook do lead (n8n)

Cole a URL do webhook em **Webhook do lead** no painel. A página faz `POST` JSON:

```json
{
  "nome": "Maria",
  "whatsapp": "(83) 9 9999-9999",
  "whatsapp_digits": "83999999999",
  "origem": "lp-live-arya",
  "empreendimento": "Arya Living",
  "evento": "Quarta, 30 de setembro 20h",
  "pagina": "https://seudominio.com/?utm_source=ig",
  "enviado_em": "2026-09-17T18:00:00.000Z",
  "utm_source": "ig", "utm_campaign": "...", "fbclid": "..."
}
```

O n8n valida o WhatsApp e responde (com `Respond to Webhook`, JSON):

- Número válido → `{ "ok": true }` → o card vira **"Vaga garantida"** com a *Mensagem de sucesso*.
  Dispare a confirmação no WhatsApp da pessoa nesse mesmo fluxo.
- Número inválido → `{ "ok": false, "message": "texto opcional" }` → o formulário continua na tela
  com um aviso vermelho (a *Mensagem de número inválido* do painel, ou o `message` que você mandar).
- Opcional: `{ "redirect": "https://seudominio.com/obrigado" }` → a página navega para lá.

O Webhook precisa estar em modo **"Respond: Using Respond to Webhook node"** para a resposta chegar.
Se o webhook cair (rede, 5xx), a página abre o WhatsApp do corretor como reserva.

Habilite CORS no nó Webhook do n8n (Allowed Origins = seu domínio, ou `*`).

## Vídeo do hero

Coloque o MP4 (mudo, 8–12 s, ≤ 5 MB, H.264 1920×1080) em `img/drone.mp4` e preencha
**URL do vídeo** com `./img/drone.mp4` — ou use uma URL pública (R2, Stream).

## Rastreamento

**Meta Pixel ID** e **Microsoft Clarity ID** no painel. Eventos: `PageView`, `Lead`,
`InscricaoLive` / `PedidoTabela`, `SectionView`, `ScrollDepth`, `PlantaZoom`.
