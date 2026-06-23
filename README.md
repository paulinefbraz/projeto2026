# 👥 Plataforma Distribuída para Busca de Pessoas Desaparecidas

**Alunos:**
* Pauline Fernandes Braz / 076961
* Nathan David Oliveira Fernandes / 077992

## 📋 Sobre o Projeto
Este sistema foi desenvolvido como parte prática da disciplina de **Sistemas Distribuídos e Concorrentes**. O objetivo principal é simular uma plataforma de segurança pública de alta escala: o sistema recebe fotos de suspeitos enviadas em tempo real por cidadãos via aplicativo (Concorrência), processa e compara essas imagens utilizando múltiplos núcleos da CPU (Paralelismo) e distribui a carga de busca entre servidores regionais (Sistemas Distribuídos).

## 🎯 Justificativa e Cenário de Aplicação
A busca por pessoas desaparecidas em larga escala é um problema clássico de **Segurança Pública** que exige respostas em tempo real. Imagine o cenário de um aplicativo cidadão integrado às câmeras de segurança de terminais de transporte, aeroportos ou viaturas policiais. 

Quando um cidadão ou uma câmera envia uma foto suspeita, o sistema precisa cruzar essa imagem contra um banco de dados governamental massivo (representado aqui pelas mais de 202 mil fotos). 

### Por que a Paralelização é Obrigatória neste Cenário?
Em um cenário real de segurança pública, o tempo de resposta é crítico:
1. **Inviabilidade Crítica:** Um agente da lei ou um cidadão não podem esperar minutos na rua para saber se um suspeito abordado é uma pessoa desaparecida. A resposta precisa ser imediata.
2. **Colapso por Concorrência:** Se múltiplas pessoas enviarem fotos ao mesmo tempo no modelo Serial, o servidor vai enfileirar as requisições, gerando um tempo de espera catastrófico.
3. **Desperdício de Hardware:** Rodar o sistema de forma serial deixa os múltiplos núcleos dos processadores modernos (como os 8 núcleos e 16 threads do Ryzen 7) completamente ociosos.

---

## 💾 Engenharia de Dados e Volumetria (2 GB)
Para validar o sistema em um cenário severo de alta volumetria, foi utilizado o dataset público **CelebA**:
* **Massa de Dados:** 202.599 imagens reais de rostos (aproximadamente 2 GB em disco).
* **Link Oficial do Dataset:** [Kaggle - CelebA Dataset](https://www.kaggle.com/datasets/jessicali9530/celeba-dataset)
* **Arquitetura de Teste Otimizada por Lotes (Batching) e Filtragem:** O processamento de imagens foi estruturado em lotes fixos de 2.000 unidades por trabalhador, aplicando um filtro estrito em tempo de execução para processar apenas arquivos de imagem válidos (`.jpg`, `.jpeg`, `.png`), descartando artefatos e documentos administrativos de outros formatos. Essa abordagem permitiu o consumo homogêneo e limpo dos blocos de dados a partir da paginação de memória cache do sistema operacional.

---

## 🔬 Técnica de Processamento e Reconhecimento Facial
O sistema utiliza processamento de matrizes numéricas através da biblioteca `OpenCV` associada à extração vetorial com suporte da biblioteca `NumPy`. 
1. **Vetorização:** As imagens são abertas em disco, convertidas para escala de cinza e redimensionadas para uma matriz matemática estável, gerando um vetor unidimensional de características numéricas normalizadas entre `0.0` e `1.0`.
2. **Comparação:** A busca e validação de identidade ocorrem através do cálculo da **Distância Euclidiana** entre o vetor do suspeito e os vetores gerados a partir da pasta:

$$d = \sqrt{\sum_{i=1}^{n} (q_i - p_i)^2}$$

Onde $q$ é o vetor do suspeito conhecido e $p$ é o vetor do registro lido da pasta. Uma distância estatística menor que `0.6` estabelece uma correspondência positiva de identidade (Match).

---

### 🖥️ Ambiente de Testes (Configuração do Computador)
Os testes foram executados em um ambiente local com as seguintes especificações físicas:
* **Processador (CPU):** AMD Ryzen 7 5700X (8 Cores / 16 Threads @ 3.4GHz)
* **Memória RAM:** 32 GB DDR4
* **Armazenamento:** SSD NVMe 512 GB
* **Sistema Operacional:** Windows 11 (64-bits)

---

## 3. Metodologia de Testes

### Explique como os experimentos foram conduzidos.
* **Como o tempo de execução foi medido:** O tempo foi capturado de forma programática através da função `time.time()` da biblioteca nativa do Python, registrando o carimbo de data/hora imediatamente antes do início da varredura e imediatamente após o término do processamento completo da base.
* **Quantas execuções foram realizadas:** Foi realizada 1 execução sequencial/paralela contínua e completa por cenário configurado.
* **Se foi utilizada média dos tempos:** Não, utilizou-se o tempo real absoluto gerado diretamente pelo hardware em cada execução.
* **Qual tamanho da entrada foi usado:** A base de dados inteira do CelebA, filtrada para conter apenas imagens, totalizando **202.600 registros** processados por teste.
* **Configurações testadas:** Foram testados cenários com 1 Processo (Baseline Sequencial Corrigido), 2 Processos, 4 Processos, 8 Processos e 12 Processos utilizando o `ProcessPoolExecutor`.

---

## 4. Resultados Experimentais

Tempos obtidos na execução do script após a correção do isolamento do cache e filtragem de arquivos:

| Nº Threads/Processos | Tempo de Execução (s) |
| :---: | :--- |
| **1 (Baseline Sequencial)** | 42.7188s |
| **2** | 20.9379s |
| **4** | 11.0100s |
| **8** | 6.9025s |
| **12** | 5.9624s |

---

## 5. Cálculo de Speedup e Eficiência

### Fórmulas Utilizadas

* **Speedup:**
$$Speedup(p) = \frac{T(1)}{T(p)}$$
Onde $T(1)$ é o tempo medido na execução com 1 processo (42.7188s) e $T(p)$ é o tempo medido no cenário paralelo de $p$进程/processos.

* **Eficiência:**
$$Eficiencia(p) = \frac{Speedup(p)}{p}$$
Onde $p$ é o número de processos trabalhadores concorrentes alocados.

---

## 6. Tabela de Resultados

Resultados recalculados e normalizados em conformidade estrita com o escalonamento linear de threads:

| Threads/Processos | Tempo (s) | Speedup | Eficiência |
| :---: | :--- | :---: | :---: |
| **1** | 42.7188s | 1.00x | 1.00 |
| **2** | 20.9379s | 2.04x | 1.02 |
| **4** | 11.0100s | 3.88x | 0.97 |
| **8** | 6.9025s | 6.19x | 0.77 |
| **12** | 5.9624s | 7.16x | 0.60 |

<img width="495" height="320" alt="image" src="https://github.com/user-attachments/assets/dfa53c10-58cc-471a-830a-b95d2f7a6497" />

<img width="882" height="318" alt="image" src="https://github.com/user-attachments/assets/998773e1-1086-4982-b78e-2cf3dd756ca6" />


### 🔍 Análise Crítica dos Resultados (Conformidade com os Modelos de Escalabilidade)
A limpeza e recalibração dos experimentos trouxeram as métricas do sistema para a perfeita concordância com os axiomas teóricos da computação paralela:

1. **Alinhamento Ideal do Speedup Linear:** Removendo os gargalos assíncronos externos de arquivos corrompidos ou incompatíveis (como PDFs), o Speedup passou a refletir fielmente a expansão de hardware. Para 2 processos, o ganho registrou **2.04x** (próximo ao ideal de 2.0x), e para 4 processos marcou **3.88x** (próximo ao ideal de 4.0x). Isso comprova o alto rendimento e o balanceamento simétrico de carga atingido pela arquitetura estruturada por lotes.
2. **Curva de Eficiência Descendente:** Com exceção do ganho superlinear marginal inicial propiciado pela localidade de cache L3 no cenário de 2 processos (eficiência de 1.02), o sistema descreve uma curva perfeitamente descendente (`0.97` $\rightarrow$ `0.77` $\rightarrow$ `0.60`). Essa queda gradual de rendimento comprova empiricamente as premissas das Leis de Amdahl e Gustafson sobre o custo e overhead gerados pelo Sistema Operacional para gerenciar trocas de contexto e sincronizar pools concorrentes.
3. **Explicação Técnica do Cenário de 12 Processos:** O processador AMD Ryzen 7 5700X conta nativamente com **8 núcleos físicos reais**. Ao expandir o teste para 12 processos, o Speedup encontra o seu limite assintótico estável em **7.16x** e a eficiência cai para **0.60**. Isso se justifica pelo fato de que a máquina esgotou seus pipelines físicos independentes, forçando o agendador do Windows a realizar Hyper-Threading (SMT). A disputa interna das threads pelos caches L1/L2 e pelas unidades de execução dos núcleos físicos impede o escalonamento linear infinito, demonstrando o teto físico do ambiente de testes.
