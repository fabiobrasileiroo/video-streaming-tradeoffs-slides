# 🎬 Streaming & Trade-offs — Monolito vs. Microsserviços

> **Exercício 5 — Análise de Trade-offs e Atributos de Qualidade de Software**

Apresentação de slides técnica e minimalista analisando a decisão arquitetural entre Monolito e Microsserviços para plataformas de streaming de vídeo.

---

## 🎯 Conteúdo do Exercício 5

Um arquiteto de software precisa decidir entre duas alternativas para um sistema de streaming de vídeo:
* **Opção A:** Monolito único, mais simples de desenvolver e testar, mas que exige reimplantação completa a cada mudança.
* **Opção B:** Arquitetura de microsserviços, que permite escalar partes do sistema independentemente, mas aumenta a complexidade de operação e comunicação entre serviços.

### 📌 Respostas Técnicas

#### (a) Quais atributos de qualidade são favorecidos por cada opção?
* **Opção A (Monolito):**
  * **Testabilidade (Testability):** Testes de integração e ponta a ponta rodam em processo (*in-memory*), sem necessidade de mocks distribuídos de rede.
  * **Desempenho & Baixa Latência (Performance / Zero-Network Hop):** Comunicação intra-processo via memória; zero sobrecarga de serialização JSON/Protobuf e handshakes TLS.
  * **Consistência Transacional (ACID):** Transações atômicas simples em banco relacional, evitando padrões Sagas e consistência eventual.
  * **Simplicidade Operacional (Operability):** Pipeline CI/CD único, menor custo total de propriedade (TCO) e depuração local trivial.
* **Opção B (Microsserviços):**
  * **Escalabilidade Granular (Resource Elasticity):** Escala nós CPU/GPU-bound de *Transcoding* apenas durante picos de upload, sem replicar o *Catálogo* ou *Auth*.
  * **Isolamento de Falhas (Availability / Bulkhead):** Falhas no serviço de recomendações ou comentários não interrompem o core de streaming de vídeo.
  * **Implantabilidade Independente (Deployability):** Squads de negócio realizam deploys contínuos sem recompilar ou retestar o pipeline de vídeo.
  * **Heterogeneidade Tecnológica (Modifiability):** Uso de stacks especializadas (C++/Rust + FFmpeg para transcoding; Go/Node.js para APIs; Python para IA).

#### (b) Qual é o principal trade-off envolvido nessa decisão?
* **Trade-off Fundamental:** `Complexidade Operacional & Consistência de Dados ⇄ Elasticidade Granular & Autonomia de Deploy`.
  * O **Monolito** troca elasticidade granular por simplicidade operacional (baixo TCO, risco de deploy concentrado).
  * Os **Microsserviços** trocam simplicidade e consistência estrita por escalabilidade e autonomia (pagando o *Microservices Premium* com tracing distribuído, latência de rede e orquestração).

#### (c) Em que cenário cada opção seria mais adequada?
* **Monolito (Opção A):** Startups, MVPs, validação de mercado, times pequenos (< 15 devs), ou quando o transcoding é delegado a serviços gerenciados (ex: AWS MediaConvert / Mux).
* **Microsserviços (Opção B):** Plataformas em hiper-escala (ex: Twitch, Netflix) com milhões de conexões simultâneas, múltiplos squads autônomos por domínio (Lei de Conway) e transcoding intensivo in-house.

---

## 🚀 Como Visualizar os Slides
Basta abrir o arquivo `index.html` em qualquer navegador ou acessar o link publicado no GitHub Pages.
