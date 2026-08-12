---
theme: excali-slide
title: 'Arquitetura de Software: Streaming & Trade-offs'
info: |
  Exercício 5 — Análise de Trade-offs: Monolito vs. Microsserviços em Streaming de Vídeo
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Streaming & Trade-offs
### Monolito vs. Microsserviços em Sistemas de Vídeo

<div class="pt-6">
  <span class="px-3 py-1 bg-amber-100 dark:bg-amber-900/40 text-amber-800 dark:text-amber-200 rounded text-sm font-mono border border-amber-300">Exercício 5 — Arquitetura de Software</span>
</div>

<div class="abs-bottom-left text-sm opacity-60 m-6 font-mono">
  Análise de Atributos de Qualidade & Decisão Arquitetural
</div>

---
layout: default
---

# System Design: Como Funciona o Streaming?

Arquitetura de processamento e distribuição de vídeo sob demanda (VOD) e Live.

```mermaid
flowchart LR
    A["🎬 Video Upload\n(Raw MP4/MOV)"] --> B["⚙️ Transcoder\n(Chunking & Bitrates)"]
    B --> C["📦 Object Storage\n(HLS/DASH .m3u8/.mpd)"]
    C --> D["🌐 CDN Edge\n(Geo Caching)"]
    D --> E["📱 Video Player\n(Adaptive Bitrate - ABR)"]
    
    subgraph ControlPlane ["🎛️ Control Plane"]
        F["🔐 Auth & DRM"]
        G["📚 Catálogo"]
        H["💳 Billing"]
    end
    
    E -.-> ControlPlane
```

<div class="grid grid-cols-3 gap-4 text-xs mt-4">
  <div class="p-3 bg-gray-100/60 dark:bg-gray-800/60 rounded border border-gray-300 dark:border-gray-700">
    <b>1. Ingestão & Chunking</b><br>
    Quebra do vídeo bruto em fragmentos temporais de 2s a 6s.
  </div>
  <div class="p-3 bg-gray-100/60 dark:bg-gray-800/60 rounded border border-gray-300 dark:border-gray-700">
    <b>2. Multi-bitrate Encoding</b><br>
    Gera resoluções (1080p, 720p, 480p) e manifestos HLS/DASH.
  </div>
  <div class="p-3 bg-gray-100/60 dark:bg-gray-800/60 rounded border border-gray-300 dark:border-gray-700">
    <b>3. Distribuição & ABR</b><br>
    CDN aproxima os chunks; player adapta a qualidade dinamicamente.
  </div>
</div>

---
layout: default
---

# O Dilema Arquitetural (Exercício 5)

Como estruturar o software de uma plataforma com cargas tão heterogêneas?

<div class="grid grid-cols-2 gap-6 mt-6">

<div class="p-4 rounded-lg bg-blue-50/50 dark:bg-blue-950/30 border border-blue-200 dark:border-blue-800">

### 🏛️ Opção A: Monolito Único
* Base de código unificada (Auth + Catálogo + Billing + Ingestão).
* Processo único de execução e banco de dados centralizado.
* **Vantagem**: Fácil de desenvolver, depurar e testar.
* **Gargalo**: Reimplantação total a cada modificação.
</div>

<div class="p-4 rounded-lg bg-emerald-50/50 dark:bg-emerald-950/30 border border-emerald-200 dark:border-emerald-800">

### 🧩 Opção B: Microsserviços
* Serviços isolados por Bounded Context (*Transcoder*, *Catalog*, *Auth*).
* Bancos de dados descentralizados e comunicação via rede/mensageria.
* **Vantagem**: Escalar componentes de forma independente.
* **Gargalo**: Complexidade operacional, observabilidade e rede.
</div>

</div>

---
layout: default
---

# (a) Atributos Favorecidos: Opção A (Monolito)

O Monolito prioriza coesão de ciclo de vida e simplicidade de desenvolvimento.

<div class="grid grid-cols-2 gap-4 text-sm mt-4">

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-blue-600 font-bold">🧪 Testabilidade (Testability)</h4>
  <p class="text-xs mt-1">Testes de ponta a ponta e integração rodam em processo (in-memory), sem necessidade de mocks distribuídos ou Testcontainers para cada microserviço.</p>
</div>

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-blue-600 font-bold">⚡ Desempenho Interno (Latency / Zero-Hop)</h4>
  <p class="text-xs mt-1">Comunicação intra-processo via chamadas de memória. Zero overhead de serialização (JSON/Protobuf), handshakes TLS e latência de rede entre módulos.</p>
</div>

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-blue-600 font-bold">🔒 Consistência de Dados (ACID)</h4>
  <p class="text-xs mt-1">Transações atômicas nativas no banco de dados. Elimina a necessidade de padrões complexos como Sagas, 2PC e reconciliação de consistência eventual.</p>
</div>

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-blue-600 font-bold">🛠️ Simplicidade Operacional (Operability)</h4>
  <p class="text-xs mt-1">Menor custo total de propriedade (TCO). Pipeline de CI/CD único, infraestrutura enxuta (1 cluster simples) e depuração local trivial.</p>
</div>

</div>

---
layout: default
---

# (a) Atributos Favorecidos: Opção B (Microsserviços)

Microsserviços favorecem elasticidade diante de cargas assimétricas.

<div class="grid grid-cols-2 gap-4 text-sm mt-4">

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-emerald-600 font-bold">📈 Escalabilidade Granular (Resource Elasticity)</h4>
  <p class="text-xs mt-1">Escala elástica de workers CPU/GPU-bound de <i>Transcoding</i> durante picos de upload, sem precisar replicar o serviço leve de <i>Catálogo</i> ou <i>Auth</i>.</p>
</div>

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-emerald-600 font-bold">🛡️ Isolamento de Falhas (Availability / Bulkhead)</h4>
  <p class="text-xs mt-1">Uma falha ou estouro de memória no serviço de Recomendações ou Comentários não derruba o Core de Reprodução e Streaming de Vídeo.</p>
</div>

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-emerald-600 font-bold">🚀 Implantabilidade Independente (Deployability)</h4>
  <p class="text-xs mt-1">Squads realizam deploys contínuos em seus domínios (ex: 30x/dia no Catálogo) sem retestar ou recompilar o pipeline de encoding de vídeo.</p>
</div>

<div class="p-3 rounded bg-gray-50 dark:bg-gray-800/50 border border-gray-200 dark:border-gray-700">
  <h4 class="text-emerald-600 font-bold">🔧 Heterogeneidade Tecnológica (Modifiability)</h4>
  <p class="text-xs mt-1">Stack poliglota sob medida: C++/Rust + FFmpeg para transcoding de alta performance, Go/Node.js para APIs de streaming e Python para IA/Recomendação.</p>
</div>

</div>

---
layout: default
---

# (b) O Principal Trade-off

O equilíbrio clássico entre **Complexidade Operacional** e **Elasticidade Granular**.

<div class="mt-6 p-4 rounded-xl bg-amber-500/10 border-2 border-amber-500/30">
  <div class="text-center font-mono text-lg font-bold text-amber-700 dark:text-amber-300">
    Complexidade de Rede & Operação <span class="text-rose-500">⇄</span> Elasticidade de Recursos & Autonomia de Deploy
  </div>
</div>

<div class="grid grid-cols-2 gap-4 text-xs mt-6">

<div class="p-3 rounded border border-gray-300 dark:border-gray-700">
  <b>No Monolito paga-se em Acoplamento:</b>
  <ul class="list-disc ml-4 mt-2 space-y-1">
    <li>Deploy All-or-Nothing (risco concentrado).</li>
    <li>Ineficiência de hardware (instâncias infladas para abrigar encoding e web).</li>
    <li>Conflito de dependências entre times.</li>
  </ul>
</div>

<div class="p-3 rounded border border-gray-300 dark:border-gray-700">
  <b>Nos Microsserviços paga-se o "Microservice Premium":</b>
  <ul class="list-disc ml-4 mt-2 space-y-1">
    <li>Latência de rede adicional (Fallacies of Distributed Computing).</li>
    <li>Complexidade de observabilidade (Distributed Tracing, OpenTelemetry).</li>
    <li>Sobrecarga de infra (Kubernetes, Service Mesh, API Gateway, Sagas).</li>
  </ul>
</div>

</div>

---
layout: default
---

# (c) Cenários de Adequação Técnica

Critérios objetivos para direcionar a decisão arquitetural:

<div class="grid grid-cols-2 gap-6 mt-4">

<div class="p-4 rounded-lg bg-blue-50/40 dark:bg-blue-950/20 border border-blue-300 dark:border-blue-800">
  <h3 class="text-blue-600 font-bold text-sm">Quando escolher a Opção A (Monolito)</h3>
  <ul class="text-xs space-y-2 mt-3">
    <li><b>Fase do Produto:</b> Startups, MVPs, plataformas corporativas internas ou validação de Product-Market Fit.</li>
    <li><b>Tamanho do Time:</b> Equipes pequenas a médias (< 15 engenheiros) trabalhando em um domínio em rápida evolução.</li>
    <li><b>Perfil de Carga:</b> Tráfego previsível, ou processamento de vídeo delegado a serviços gerenciados (ex: AWS MediaConvert / Mux).</li>
    <li><b>Foco:</b> Maximizar <i>Time-to-Market</i> e minimizar custos de infraestrutura e governança.</li>
  </ul>
</div>

<div class="p-4 rounded-lg bg-emerald-50/40 dark:bg-emerald-950/20 border border-emerald-300 dark:border-emerald-800">
  <h3 class="text-emerald-600 font-bold text-sm">Quando escolher a Opção B (Microsserviços)</h3>
  <ul class="text-xs space-y-2 mt-3">
    <li><b>Fase do Produto:</b> Plataformas em hiper-escala (ex: Netflix, Twitch) com milhões de visualizadores simultâneos (CCU).</li>
    <li><b>Estrutura Organizacional:</b> Múltiplos squads autônomos (Lei de Conway) por Bounded Context (Transcoding, DRM, Ads, Billing).</li>
    <li><b>Assimetria Extrema:</b> Ingestão/Transcoding massivo in-house exigindo clusters elásticos de GPUs dedicadas.</li>
    <li><b>Foco:</b> Resiliência de missão crítica (Zero-Downtime) e deploys desacoplados em alta frequência.</li>
  </ul>
</div>

</div>

---
layout: default
---

# Síntese & Recomendação Técnica

Na prática da engenharia de software moderna, a escolha não precisa ser binária.

<div class="p-4 rounded-lg bg-purple-50/40 dark:bg-purple-950/30 border border-purple-300 dark:border-purple-700 mt-4">
  <h3 class="text-purple-600 dark:text-purple-300 font-bold text-sm">💡 Estratégia Recomendada: Modular Monolith + Async Workers</h3>
  <p class="text-xs mt-2 text-gray-700 dark:text-gray-300 leading-relaxed">
    Iniciar com um <b>Monolito Modular</b> bem estruturado para o Control Plane (Auth, Catálogo, Billing) e isolar apenas o gargalo computacional de <b>Transcoding/Ingestão</b> em workers assíncronos desacoplados por filas (Kafka/SQS).
  </p>
  <p class="text-xs mt-2 text-gray-700 dark:text-gray-300 leading-relaxed">
    Conforme o produto ganha escala e múltiplos times, utiliza-se o padrão <b>Strangler Fig</b> para extrair serviços específicos sob demanda comprovada de carga e governança.
  </p>
</div>

<div class="grid grid-cols-3 gap-3 text-center text-xs mt-6">
  <div class="p-2 border rounded"><b>1. Simplicidade Primeiro</b><br><span class="opacity-70">Prove o valor do negócio</span></div>
  <div class="p-2 border rounded"><b>2. Isole Gargalos Reais</b><br><span class="opacity-70">Transcoding assíncrono</span></div>
  <div class="p-2 border rounded"><b>3. Distribua com Dados</b><br><span class="opacity-70">Evite distribuição prematura</span></div>
</div>
