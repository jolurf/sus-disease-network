# Mapa de doenças do SUS · SUS disease network

**Site:** https://jolurf.github.io/sus-disease-network/

Rede de doenças construída a partir dos registros ambulatoriais especializados do SUS
(SIA), residentes no Brasil, 2014–2023. Cada nó é uma categoria da CID-10 (3 caracteres).
Duas doenças se ligam quando aparecem no mesmo paciente mais do que o acaso explica. O site
deixa trocar a população do grafo (geral, sexo, faixa etária no evento, região, raça/cor),
buscar doenças e baixar as tabelas de doenças e de ligações de cada grafo em CSV. A página
[Como o mapa é feito](https://jolurf.github.io/sus-disease-network/metodo.html) traz o
método em linguagem não técnica.

> **Versão preliminar.** Os resultados ainda não foram revisados por pares e as análises de
> sensibilidade estão em andamento; números podem mudar. Associação no registro
> administrativo não é causalidade.

**Em números (grafo geral):** 174,8 milhões de pacientes passaram pela auditoria de
identidade, e 67,5 milhões deles têm ao menos um CID de doença. O grafo tem 1 435 doenças,
102 242 ligações, 25 170 ligações com direção temporal e 13 comunidades.

## Metodologia

### Fonte
- **Base:** microdados públicos do Sistema de Informações Ambulatoriais (SIA/SUS), DATASUS.
- **Período:** atendimentos de 2014 a 2023. Os anos de 2012 e 2013 servem só para reconhecer
  doenças que o paciente já tinha antes do período (washout).
- **Identidade:** o paciente é reconhecido por um **identificador único criptografado**,
  presente nos microdados públicos.
- **Diagnósticos:** vêm do BPA-I e das **APAC** (medicamentos de alto custo, quimioterapia,
  radioterapia e demais procedimentos), ligados ao paciente pelo mesmo identificador.
- **O que não entra:** o BPA consolidado (PA), que não identifica o paciente.

### Critérios de entrada do paciente
1. **Identificador consistente:** passa por uma auditoria de qualidade. Saem os
   identificadores com sinais de que correspondem a mais de uma pessoa, porque misturar duas
   pessoas criaria ligações falsas.
2. **Dados demográficos plausíveis**, com remoção de valores usados como preenchimento padrão
   pelos sistemas.
3. **Residência dentro do recorte:** o município de residência é o mais frequente nos registros
   do paciente.
4. **Pelo menos um CID de doença no período.**

A exclusão por inconsistência é maior em quem tem muitos registros: 0,8% em quem tem
1 registro e 10,5% em quem tem 100 ou mais. Pacientes com muita multimorbidade ficam um pouco
sub-representados.

### O que conta como doença
- **Unidade:** a categoria da CID-10 com 3 caracteres (ex.: `I10`, hipertensão).
- **Excluídos:** sintomas (R), causas externas (V–Y), fatores de contato com serviços (Z) e
  códigos especiais (U).
- **Também excluídos:** `A00` (cólera) e `A01` (febres tifoide e paratifoide). São o primeiro
  código da tabela e aparecem como preenchimento padrão, sobretudo no exame anatomopatológico,
  que passou a exigir CID em 2018 (Portaria SAS 643/2018).
- **Primeira ocorrência:** o mês do primeiro registro da doença no SIA. É o primeiro registro,
  não o início da doença.

### Ligações entre doenças
Para cada par de doenças com pelo menos 100 pacientes cada:

| medida | o que diz |
|---|---|
| **RR** | razão entre pacientes observados com as duas doenças e o esperado se fossem independentes (IC de Garwood) |
| **φ** | correlação entre ter uma e ter a outra; é o peso usado para desenhar e agrupar |
| **teste** | exato de Fisher, com correção de Benjamini–Hochberg a 1% sobre **todos** os pares possíveis |

Com dezenas de milhões de pacientes, quase todo par sai significativo. Por isso a ligação só
entra se:
- o **limite inferior do IC do RR for maior que 1**;
- **pelo menos 20 pacientes** tiverem as duas doenças.

Cada ligação também é comparada ao esperado por faixa de idade (5 anos) × sexo (padronização
indireta). Uma coluna marca as que se explicam só por as duas doenças serem comuns na mesma
idade.

**Artefato de serviço.** Uma ligação sai do mapa quando é padrão de codificação de poucos
estabelecimentos. As duas condições têm de valer juntas:
1. mais da metade dos pacientes do par veio dos 3 estabelecimentos (CNES) que mais o registram;
2. fora desses estabelecimentos, o limite superior do IC do RR fica abaixo de 1,25, ou seja, a
   associação some.

Pouco dado fora dos 3 serviços não basta para retirar a ligação. Isso protege doenças tratadas
em centros de referência. Pela regra saem 1 824 ligações (1,75%), e uma exclusão manual
documentada (C40 × G46) cobre um padrão espalhado por vários serviços de fisioterapia.

### Direção temporal
- **Quem entra:** os pacientes que tiveram as duas doenças registradas pela primeira vez com
  **1 a 60 meses** de distância.
- **Teste:** compara quantos tiveram cada ordem (teste binomial, com correção de
  Benjamini–Hochberg).
- **O que não dá direção:** doenças registradas no mesmo mês, e doenças já presentes em
  2012–2013 (prevalentes).

### Comunidades e pontes
- **Comunidades:** algoritmo de Leiden com modularidade de Newman (resolução γ = 1),
  ponderado por φ. Os rótulos mostrados são as 3 doenças centrais de cada grupo.
- **Pontes:** betweenness com distância 1/φ, e participação de Guimerà–Amaral (o quanto as
  ligações de uma doença se dividem entre comunidades).
- **Gatilho ↔ desfecho:** de −1 (só inicia trajetórias) a +1 (só encerra).

### Populações (estratos)
- **Sexo**, **raça/cor** e **região de residência** (pela UF mais frequente) do paciente.
- **Faixa etária pela idade no evento** (0–17, 18–39, 40–59, 60–79, 80+). Cada doença entra
  na faixa em que foi registrada pela primeira vez, então um paciente pode contribuir para mais
  de uma faixa, com as doenças de cada fase da vida.

### Limites
- O SIA é **atenção especializada**. A atenção primária não está nele, então diabetes e
  hipertensão costumam aparecer tarde, muitas vezes já com a complicação.
- Uma pessoa com mais de um identificador vira duas pessoas, o que enfraquece as ligações.
- A **cobertura de CID varia por região**: cerca de 23% dos pacientes no Norte têm CID de
  doença, contra 31% no Sudeste. Uma rede regional mais pobre pode refletir registro, não
  epidemiologia.
- A raça/cor **"Amarela"** é registrada em excesso no SIA (até 17% dos registros em alguns
  meses, contra cerca de 1% na população) e não deve ser lida como tal.
- Códigos residuais (I98, I97, N39, C80) aparecem como pontes por serem genéricos, não
  necessariamente por papel clínico.

## Dados publicados
- Só **contagens agregadas**: nenhuma linha corresponde a uma pessoa.
- Doenças entram com pelo menos 100 pacientes, e ligações com pelo menos 20 pacientes que
  tiveram as duas doenças.
- `tabelas/<grafo>_doencas.csv`: CID, descrição, capítulo, comunidade, pacientes, prevalência
  por mil, grau, betweenness, participação e índice gatilho/desfecho.
- `tabelas/<grafo>_ligacoes.csv`: par de CIDs, pacientes com as duas, RR com IC, φ, RR
  padronizado, p ajustado e a marca de "explicável por idade e sexo".

## Uso local
O site carrega os dados por `fetch`, então precisa de um servidor:

```bash
python -m http.server 8000   # e abra http://localhost:8000
```

## Referências
- Hidalgo CA et al. A dynamic network approach for the study of human phenotypes.
  *PLoS Comput Biol* 2009;5:e1000353.
- Jensen AB et al. Temporal disease trajectories condensed from population-wide registry data
  covering 6.2 million patients. *Nat Commun* 2014;5:4022.
- Traag VA, Waltman L, van Eck NJ. From Louvain to Leiden: guaranteeing well-connected
  communities. *Sci Rep* 2019;9:5233.

---

*English.* Disease network built from Brazil's public outpatient specialty-care records
(SUS/SIA), 2014–2023, using ICD-10 3-character categories, with strata by sex, age at event,
region and race/skin colour.
- **Patients:** encrypted unique identifier that passes a consistency audit; identifiers that
  appear to merge more than one person are excluded.
- **Links:** Fisher's exact test with Benjamini–Hochberg correction, lower CI of the relative
  risk above 1, and at least 20 co-occurring patients. Service-coding artefacts are removed.
- **Direction:** binomial test on first-record order, 1–60 months apart.
- **Communities:** Leiden.

Aggregated counts only. Preliminary, not yet peer reviewed.
