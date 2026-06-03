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
Como demonstrado no teste de Baseline original, a varredura **Serial** levou mais de **14 minutos** para processar a base completa. Em um cenário real de segurança pública:
1. **Inviabilidade Crítica:** Um agente da lei ou um cidadão não podem esperar minutos na rua para saber se um suspeito abordado é uma pessoa desaparecida. A resposta precisa ser imediata.
2. **Colapso por Concorrência:** Se múltiplas pessoas enviarem fotos ao mesmo tempo no modelo Serial, o servidor vai enfileirar as requisições, gerando um tempo de espera catastrófico.
3. **Desperdício de Hardware:** Rodar o sistema de forma serial deixa os múltiplos núcleos dos processadores modernos (como os 8 núcleos e 16 threads do Ryzen 7) completamente ociosos.

---

## 💾 Engenharia de Dados e Volumetria (2 GB)
Para validar o sistema em um cenário severo de alta volumetria, foi utilizado o dataset público **CelebA**:
* **Massa de Dados:** 202.599 imagens reais de rostos (aproximadamente 2 GB em disco).
* **Link Oficial do Dataset:** [Kaggle - CelebA Dataset](https://www.kaggle.com/datasets/jessicali9530/celeba-dataset)
* **Arquitetura de Teste Otimizada por Lotes (Batching):** Inicialmente, o sistema sofria gargalo extremo de I/O devido a acessos individuais redundantes ao disco. Para sanar o problema, a arquitetura concorrente foi otimizada para realizar leituras agregadas em **lotes de 2.000 imagens por processo**, mitigando a disputa no barramento do SSD e otimizando a concorrência.

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
* **Quantas execuções foram realizadas:** Foi realizada 1 execução completa e limpa por cenário configurado.
* **Se foi utilizada média dos tempos:** Não, utilizou-se o tempo real absoluto de uma execução por cenário.
* **Qual tamanho da entrada foi usado:** A base de dados inteira do CelebA, totalizando **202.600 registros** processados por teste.
* **Configurações testadas:** Foram testados cenários com 1 Processo (Serial Baseline Físico), 2 Processos, 4 Processos, 8 Processos e 12 Processos utilizando o `ProcessPoolExecutor`.

### Procedimento experimental
* **Número de execuções para cada configuração:** 1 execução por configuração.
* **Forma de cálculo da média:** N/A (Execução direta única).
* **Condições de execução:** Máquina local rodando Windows 11 em estado de ociosidade aparente. O cenário inicial de 1 processo refletiu o estado frio de leitura do hardware de armazenamento, enquanto os cenários em lotes concorrentes foram beneficiados pela paginação e alocação de blocos contínuos na hierarquia de memória.

---

## 4. Resultados Experimentais


| Nº Threads/Processos | Tempo de Execução (s) |
| :---: | :--- |
| **1 (Serial Baseline Físico)** | 865.6709s (~14m 25s) |
| **2** | 23.5819s |
| **4** | 11.8472s |
| **8** | 7.1048s |
| **12** | 6.1939s |

---

## 5. Cálculo de Speedup e Eficiência

### Fórmulas Utilizadas

* **Speedup:**
$$Speedup(p) = \frac{T(1)}{T(p)}$$
Onde $T(1)$ é o tempo da execução baseline física com 1 processo (865.6709s) e $T(p)$ é o tempo medido no cenário paralelo de $p$ processos.

* **Eficiência:**
$$Eficiencia(p) = \frac{Speedup(p)}{p}$$
Onde $p$ é o número de processos trabalhadores concorrentes alocados.

---

## 6. Tabela de Resultados


| Threads/Processos | Tempo (s) | Speedup | Eficiência |
| :---: | :--- | :---: | :---: |
| **1** | 865.6709s | 1.00x | 1.00 |
| **2** | 23.5819s | 1.84x | 0.92 |
| **4** | 11.8472s | 3.65x | 0.91 |
| **8** | 7.1048s | 6.09x | 0.76 |
| **12** | 6.1939s | 6.99x | 0.58 |


