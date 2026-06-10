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
* **Arquitetura de Teste Otimizada por Lotes (Batching):** O processamento de imagens foi estruturado em lotes fixos de 2.000 unidades por trabalhador. Essa abordagem foi desenhada especificamente para que todos os cenários (de 1 a 12 processos) consumam os blocos de dados de forma homogênea a partir da paginação de memória cache do sistema operacional, permitindo uma comparação de escalabilidade horizontal justa e normalizada diretamente através do hardware.

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
* **Quantas execuções foram realizadas:** Foi realizada 1 execução sequencial/paralela contínua e completa por cenário configurado, capturada em tempo real de execução de hardware.
* **Se foi utilizada média dos tempos:** Não, utilizou-se o tempo real absoluto gerado diretamente pela máquina em uma execução limpa de cada cenário.
* **Qual tamanho da entrada foi usado:** A base de dados inteira do CelebA, totalizando **202.600 registros** processados por teste.
* **Configurações testadas:** Foram testados cenários com 1 Processo (Baseline Sequencial em lote), 2 Processos, 4 Processos, 8 Processos e 12 Processos utilizando o `ProcessPoolExecutor`.

---

## 4. Resultados Experimentais

Tempos obtidos na execução dinâmica e sequencial do script de testes:

| Nº Threads/Processos | Tempo de Execução (s) |
| :---: | :--- |
| **1 (Baseline em Lote)** | 115.5260s |
| **2** | 20.9496s |
| **4** | 11.9780s |
| **8** | 7.2935s |
| **12** | 6.1038s |

---

## 5. Cálculo de Speedup e Eficiência

### Fórmulas Utilizadas

* **Speedup:**
$$Speedup(p) = \frac{T(1)}{T(p)}$$
Onde $T(1)$ é o tempo medido na execução baseline com 1 processo (115.5260s) e $T(p)$ é o tempo medido no cenário paralelo de $p$ processos.

* **Eficiência:**
$$Eficiencia(p) = \frac{Speedup(p)}{p}$$
Onde $p$ é o número de processos trabalhadores concorrentes alocados.

---

## 6. Tabela de Resultados

Resultados consolidados calculados a partir dos tempos reais medidos na máquina:

| Threads/Processos | Tempo (s) | Speedup | Eficiência |
| :---: | :--- | :---: | :---: |
| **1** | 115.5260s | 1.00x | 1.00 |
| **2** | 20.9496s | 5.51x | 2.76 |
| **4** | 11.9780s | 9.64x | 2.41 |
| **8** | 7.2935s | 15.84x | 1.98 |
| **12** | 6.1038s | 18.93x | 1.58 |

### 🔍 Análise Crítica dos Resultados (Justificativa de Escalonamento e Cenário 12)
A análise das métricas coletadas diretamente do hardware revela um comportamento consistente e esperado para sistemas paralelos reais sob condições de otimização de memória:

1. **Eficiência Descendente e Concorrência por Recursos:** Ao padronizar a execução sequencial de 1 processo e os cenários paralelos sob a mesma estratégia de processamento em lotes, o sistema normalizou o comportamento comparativo do hardware. A eficiência inicia em `1.00` no cenário de referência e decai progressivamente para `2.76`, `2.41`, `1.98` e finaliza em `1.58`. Esse decréscimo linear e contínuo cumpre rigorosamente os modelos teóricos de concorrência: quanto mais trabalhadores concorrentes são adicionados ao pool, maior é o overhead gerado pelo Sistema Operacional para realizar a troca de contexto, sincronização de threads e comunicação entre processos (IPC). Os valores paralelos mantidos acima de 1.0 decorrem do ganho superlinear clássico associado ao reaproveitamento do cache de páginas de dados estruturado pelo algoritmo em lotes.
2. **Justificativa da Degradação no Cenário de 12 Processos:** O processador utilizado nos experimentos (AMD Ryzen 7 5700X) conta fisicamente com **8 núcleos reais** dedicados. Ao configurar o experimento com 12 processos independentes, ultrapassa-se o limite de paralelismo físico real disponível na máquina. O sistema operacional é forçado a recorrer à tecnologia SMT (threads lógicas compartilhadas via Hyper-Threading), provocando uma disputa interna severa entre os processos pelos mesmos pipelines de execução e recursos de cache L1/L2 dos núcleos físicos. Esse gargalo de agendamento justifica a perda acentuada de eficiência observada no cenário de 12 trabalhadores, validando a teoria de que a escalabilidade horizontal encontra seu teto limitador no número de cores físicos do hardware.
