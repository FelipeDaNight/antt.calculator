# Calculadora de piso mínimo de frete ANTT

Página única (HTML + JavaScript, sem dependências) que calcula o piso mínimo de frete da ANTT para o transporte rodoviário de cargas.

## Funcionalidades

- Escolha de eixos carregados (2 a 9) e do tipo de carga (os 12 tipos da Tabela A).
- Roteiro com origem e várias entregas, com busca de cidades pelos 5.570 municípios do Brasil (sem precisar de acento; dá para digitar a UF junto, ex.: `Morrinhos GO`).
- Km de cada trecho pela estrada, calculado pelo roteirizador gratuito do OpenStreetMap (OSRM). Opcionalmente, rota de caminhão pelo OpenRouteService (perfil `driving-hgv`) com uma chave gratuita.
- Se o roteirizador não responder, usa uma estimativa (linha reta × fator de estrada, padrão 1,25). Sempre dá para digitar o km real do trecho.
- Link para conferir a rota completa no Google Maps.
- **Pedágio estimado** (opcional, com chave do Google Maps Routes API): o Google devolve o valor para carro e o site multiplica pelo número de eixos. Com a chave, o km também passa a vir da rota do Google Maps, a mesma referência da ANTT. O valor exato do vale-pedágio fica com a IPEF no CIOT.
- Comparação com o frete combinado: avisa quando o valor está abaixo do piso (o CIOT é bloqueado em carga lotação).
- Enquadramento: 1 entrega = carga lotação (piso obrigatório); várias entregas = valor de referência.
- Botão para copiar um resumo da viagem.

## Fórmula

```
Piso = distância (km) × CCD + CC
```

- **CCD**: coeficiente de deslocamento (R$/km).
- **CC**: coeficiente de carga e descarga (R$, uma vez por viagem).

Coeficientes da **Tabela A** do Anexo II da Resolução ANTT nº 5.867/2020, com os valores da **Portaria SUROC nº 22/2026** (em vigor desde 30/09/2026, diesel S10 de referência R$ 7,33/L). Ficam no objeto `T` no início do script de `index.html`. Quando a ANTT publicar uma nova tabela, basta atualizar esses números.

## Arquivos

- `index.html`: a calculadora.
- `municipios.json`: lista de municípios (nome, UF, latitude, longitude), gerada a partir do conjunto de dados [kelvins/municipios-brasileiros](https://github.com/kelvins/municipios-brasileiros) (licença MIT).

## Como usar

A página carrega `municipios.json` por `fetch`, então precisa ser servida por HTTP (abrir o arquivo direto pelo navegador não carrega as cidades). Para testar localmente:

```
python3 -m http.server 8000
```

e abrir `http://localhost:8000`.

Para publicar de graça, ative o **GitHub Pages** no repositório (Settings → Pages → Deploy from a branch → `main` / root).

## Aviso

A rota vai do centro de uma cidade ao centro da outra, não do endereço exato da fábrica ou do cliente. O servidor público do OSRM é para uso leve; para muito volume, use a chave do OpenRouteService. A ANTT confere o piso pela rota declarada no MDF-e usando roteirizadores públicos, então confirme o km real antes de fechar o frete. Ferramenta de apoio; para casos de dúvida, use a calculadora oficial em calculadorafrete.antt.gov.br.

## Chave do Google (pedágio)

1. No Google Cloud Console, crie um projeto, ative a **Routes API** e crie uma **chave de API** (precisa de conta de faturamento; consultas com pedágio têm 1.000 grátis por mês).
2. Restrinja a chave: em *Restrições de aplicativo*, escolha *Referenciadores HTTP* e coloque `https://felipedanight.github.io/*`; em *Restrições de API*, deixe só a Routes API.
3. Em *Cotas* da Routes API, limite as consultas por dia (por exemplo, 30) para não passar da cota grátis.
4. Cole a chave no site (seção "Pedágio e km do Google") ou em `GOOGLE_KEY`, no início do script de `index.html`, para valer para todos.
