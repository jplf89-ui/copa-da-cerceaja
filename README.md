# 🍺 Copa das Cervejas 2026

Landing page de venda de kits de cervejas temáticos para a Copa do Mundo 2026.  
Kits fixos por continente + montador personalizado com cálculo em tempo real.

---

## Estrutura do projeto

```
copa-cervejas/
├── index.html                  ← página principal (landing page)
├── obrigado.html               ← página pós-pagamento aprovado
├── politica-de-privacidade.html
├── api/
│   ├── pedido.js               ← cria preferência MP + envia e-mail
│   └── webhook.js              ← recebe confirmação MP + emite NF-e
├── .env.example                ← variáveis necessárias (copie para .env)
├── vercel.json                 ← config de headers e rotas
└── package.json
```

---

## 1. GitHub

```bash
git init
git add .
git commit -m "init: copa das cervejas 2026"
git remote add origin https://github.com/SEU_USUARIO/copa-cervejas.git
git push -u origin main
```

---

## 2. Vercel

1. Acesse [vercel.com](https://vercel.com) → **Add New Project**
2. Importe o repositório do GitHub
3. **Framework Preset:** Other (não é Next.js)
4. Clique em **Deploy**

O Vercel detecta automaticamente as funções em `/api/*.js`.

### Variáveis de ambiente no Vercel

No painel do projeto → **Settings → Environment Variables**, adicione todas as variáveis do `.env.example`:

| Variável | Onde obter |
|---|---|
| `MP_ACCESS_TOKEN` | [mercadopago.com.br/developers](https://www.mercadopago.com.br/developers/panel) → Credenciais de Produção |
| `MP_PUBLIC_KEY` | Mesmo local |
| `RESEND_API_KEY` | [resend.com/api-keys](https://resend.com/api-keys) |
| `EMAIL_FROM` | Domínio verificado no Resend (ex: `pedidos@seudominio.com.br`) |
| `EMAIL_ADMIN` | Seu e-mail pessoal |
| `FOCUS_TOKEN` | [app.focusnfe.com.br](https://app.focusnfe.com.br) → API Tokens |
| `FOCUS_ENV` | `homologacao` para testes, `producao` para emitir de verdade |
| `EMPRESA_CNPJ` | Seu CNPJ (sem pontos/traços) |
| `NEXT_PUBLIC_BASE_URL` | `https://seudominio.com.br` |

---

## 3. Domínio no Registro.br → Vercel

### No painel da Vercel

1. Vá em **Settings → Domains**
2. Adicione seu domínio (ex: `copadascervejas.com.br`)
3. A Vercel mostra dois registros DNS para configurar

### No Registro.br

1. Acesse [registro.br](https://registro.br) → seu domínio → **DNS**
2. Adicione os registros:

| Tipo | Nome | Valor |
|---|---|---|
| `A` | `@` (raiz) | `76.76.21.21` |
| `CNAME` | `www` | `cname.vercel-dns.com` |

3. Aguarde 5–30 minutos para propagação
4. SSL (HTTPS) é gerado automaticamente pela Vercel

> Se for **subdomínio** (ex: `cervejas.seudominio.com.br`), adicione apenas o CNAME apontando para `cname.vercel-dns.com`.

---

## 4. Mercado Pago — Checkout Transparente

### Conta e credenciais

1. Crie conta em [mercadopago.com.br](https://www.mercadopago.com.br)
2. Acesse [Developers → Painel](https://www.mercadopago.com.br/developers/panel)
3. Crie uma **aplicação**
4. Copie **Access Token de Produção** e **Public Key** para o `.env`

### Webhook (notificações de pagamento)

No painel do MP → **Notificações Webhook**:
- URL: `https://seudominio.com.br/api/webhook`
- Eventos: `payment`

O webhook é chamado automaticamente quando um pagamento é aprovado, recusado ou pendente.

### Teste antes de ir para produção

Use as **credenciais de teste** (Sandbox) do MP com cartões de teste:
- [Cartões de teste MP](https://www.mercadopago.com.br/developers/pt/docs/checkout-api/additional-content/your-integrations/test/cards)

Troque `FOCUS_ENV=homologacao` para não emitir NFs reais durante os testes.

---

## 5. E-mail com Resend

1. Crie conta em [resend.com](https://resend.com)
2. Adicione seu domínio em **Domains** e verifique os registros DNS (eles mostram os registros para adicionar no Registro.br)
3. Crie uma API Key e adicione no `.env`
4. O `EMAIL_FROM` deve ser um e-mail do domínio verificado (ex: `pedidos@seudominio.com.br`)

> **Plano gratuito do Resend:** 3.000 e-mails/mês — mais que suficiente para começar.

---

## 6. Nota Fiscal com Focus NFe

### O que é emitido

Ao receber confirmação de pagamento aprovado pelo webhook do MP, o sistema emite automaticamente uma **NFC-e (Nota Fiscal Consumidor Eletrônica)** e envia o DANFE (PDF) por e-mail para o cliente.

### Setup

1. Crie conta em [focusnfe.com.br](https://focusnfe.com.br)
2. Em **homologação**, teste livremente sem custos
3. Configure seu certificado digital A1 no painel do Focus
4. Ajuste os dados tributários no `api/webhook.js` com seu contador:
   - `cfop` — código fiscal (5102 para venda no estado, 6102 para outros estados)
   - `icms_situacao_tributaria` — depende do seu regime (Simples, Lucro Presumido, etc.)

> **⚠️ Importante:** consulte um contador antes de ir para produção. Os códigos tributários variam conforme seu regime fiscal e o produto vendido (bebidas têm tributação específica).

### Custo Focus NFe

- Plano básico: ~R$ 49/mês com X NFs inclusas
- Consulte valores atualizados em [focusnfe.com.br/planos](https://focusnfe.com.br/planos)

---

## 7. Integrar o formulário com a API

No `index.html`, a função `enviarPedido()` está pronta para ser conectada. Substitua o `alert()` atual por:

```javascript
async function enviarPedido() {
  const nome    = document.getElementById('f-nome').value.trim();
  const wpp     = document.getElementById('f-wpp').value.trim();
  const email   = document.getElementById('f-email').value.trim();
  const kit     = document.getElementById('f-kit').value;
  const entrega = document.getElementById('f-entrega').value;
  const cep     = document.getElementById('f-cep').value.trim();
  const obs     = document.getElementById('f-obs').value.trim();

  if (!nome || !wpp || !kit) {
    alert('Preencha nome, WhatsApp e o kit escolhido.');
    return;
  }

  // Monta itens do kit personalizado se for o caso
  const itens = carrinho.map(i => ({
    nome: `${i.pais} — ${i.nome}`,
    quantidade: 1,
    preco: i.precoVenda,
  }));

  try {
    const res = await fetch('/api/pedido', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ nome, whatsapp: wpp, email, kit, itens, entrega, cep, observacoes: obs }),
    });

    const data = await res.json();

    if (data.checkout_url) {
      // Redireciona para o checkout do Mercado Pago
      window.location.href = data.checkout_url;
    } else {
      alert('Erro ao processar pedido. Entre em contato pelo WhatsApp.');
    }
  } catch (e) {
    alert('Erro de conexão. Tente novamente.');
  }
}
```

---

## 8. Fluxo completo

```
Cliente preenche formulário
  ↓
POST /api/pedido
  ↓ cria preferência no Mercado Pago
  ↓ envia e-mail de confirmação para admin + cliente
  ↓ retorna checkout_url
  ↓
Cliente é redirecionado para o checkout do MP
  ↓
Pagamento aprovado → MP chama POST /api/webhook
  ↓ busca detalhes do pagamento na API do MP
  ↓ emite NFC-e via Focus NFe
  ↓ envia DANFE por e-mail para o cliente
  ↓ envia e-mail de aviso para admin
  ↓
Cliente vê /obrigado.html
```

---

## 9. Checklist de produção

- [ ] Variáveis de ambiente configuradas no Vercel
- [ ] Domínio apontado e SSL ativo
- [ ] Domínio verificado no Resend
- [ ] Webhook do MP configurado com a URL de produção
- [ ] Credenciais de **produção** do MP (não sandbox)
- [ ] `FOCUS_ENV=producao` (após testes com homologação)
- [ ] Certificado digital A1 configurado no Focus NFe
- [ ] Dados fiscais revisados com contador
- [ ] WhatsApp atualizado no `index.html` (buscar por `5541999999999`)
- [ ] Teste completo de pedido com pagamento real de R$ 1,00
- [ ] Link de Política de Privacidade no rodapé do `index.html`

---

## Licença

Uso privado. Todos os direitos reservados.
