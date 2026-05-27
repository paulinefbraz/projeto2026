# 👥 Plataforma Distribuída para Busca de Pessoas Desaparecidas

**Alunos:**
* Pauline Fernandes Braz / 076961
* Nathan David Oliveira Fernandes / 077992

## 📋 Sobre o Projeto
Este sistema foi desenvolvido como parte prática da disciplina de **Sistemas Distribuídos e Concorrentes**. O objetivo principal é simular uma plataforma de segurança pública de alta escala: o sistema recebe fotos de suspeitos enviadas em tempo real por cidadãos via aplicativo (Concorrência), processa e compara essas imagens utilizando múltiplos núcleos da CPU (Paralelismo) e distribui a carga de busca entre servidores regionais (Sistemas Distribuídos).

---

## 💾 Engenharia de Dados e Volumetria (2 GB)
Para validar o sistema em um cenário severo de alta volumetria, foi utilizado o dataset público **CelebA**:
* **Massa de Dados:** 202.599 imagens reais de rostos (aproximadamente 2 GB em disco).
* **Link Oficial do Dataset:** [Kaggle - CelebA Dataset](https://www.kaggle.com/datasets/jessicali9530/celeba-dataset)
* **Arquitetura de Teste (IO Bound):** Para medir o impacto real de um sistema sem indexação prévia, a busca realiza uma varredura abrindo as imagens físicas direto do *Storage* (diretório local de arquivos) e extraindo as matrizes de pixels em tempo de execução, gerando o maior gargalo de leitura de disco possível.

---

## 🔬 Técnica de Processamento e Reconhecimento Facial
O sistema utiliza processamento de matrizes numéricas através da biblioteca `OpenCV` associada à extração vetorial com suporte da biblioteca `NumPy`. 
1. **Vetorização:** As imagens são abertas em disco, convertidas para escala de cinza e redimensionadas para uma matriz matemática estável, gerando um vetor unidimensional de características numéricas normalizadas entre `0.0` e `1.0`.
2. **Comparação:** A busca e validação de identidade ocorrem através do cálculo da **Distância Euclidiana** entre o vetor do suspeito e os vetores gerados a partir da pasta:

$$d = \sqrt{\sum_{i=1}^{n} (q_i - p_i)^2}$$

Onde $q$ é o vetor do suspeito capturado na rua e $p$ é o vetor do registro lido da pasta. Uma distância estatística menor que `0.6` estabelece uma correspondência positiva de identidade (Match).

---

## 🧪 Cenário de Teste Atual: Varredura Serial (Baseline)
Nesta primeira etapa do projeto, o algoritmo foi executado em modo **Serial Monothread** (utilizando estritamente 1 único núcleo do processador sem concorrência) para servir como *Baseline* (linha de base de performance) para as futuras otimizações concorrentes.

### Configuração do Pior Cenário ($O(n)$):
Para simular o cenário mais crítico do algoritmo — onde o sistema precisa varrer a base de dados inteira —, a foto de cadastro do alvo (`nathan_cadastro.jpg`) foi posicionada **estritamente na última posição** da lista de mais de 202 mil registros em disco. Uma imagem secundária (`nathan_suspeito.jpg`) foi submetida como a requisição de busca do aplicativo.

### ⏱️ Resultados Obtidos no Ambiente Local

| Métrica Avaliada | Resultado Registrado (Leitura em Disco Bruto) |
| :--- | :--- |
| **Total de Fotos (CelebA)** | 202.599 fotos |
| **Alvo Inserido no Banco** | 1 foto (`nathan_cadastro.jpg`) |
| **Volume Total de Busca** | 202.600 registros |
| **Posição do Alvo** | Último elemento da pasta |
| **Status do Reconhecimento Facial** | **🎯 SUCESSO (Alvo Localizado)** |
| **Tempo Total de Varredura Serial** | **865.6709 segundos** (~14 minutos e 25 segundos) |

---

### 🖥️ Ambiente de Testes (Configuração do Computador)
Os testes foram executados em um ambiente local com as seguintes especificações físicas:
* **Processador (CPU):** AMD Ryzen 7 5700X (8 Cores / 16 Threads @ 3.4GHz)
* **Memória RAM:** 32 GB DDR4
* **Placa-Mãe:** ASRock B450M Pro4
* **Placa de Vídeo (GPU):** NVIDIA GeForce RTX 3070 (8 GB Dedicados)
* **Armazenamento:** SSD NVMe 512 GB
* **Sistema Operacional:** Windows 11 (64-bits)

---

## 🚀 Próximos Passos do Desenvolvimento Concorrente
O teste serial físico revelou um gargalo crítico de **14 minutos e 25 segundos** quando o sistema precisa abrir arquivo por arquivo em disco usando uma única thread, tornando o sistema totalmente inviável para segurança pública em tempo real.

Na próxima fase do projeto, implementaremos:
* **Paralelismo de Dados (Multi-threading):** Divisão da carga de leitura de disco e cálculo de matrizes em fatias iguais (*chunks*), distribuindo o processamento entre as 16 threads lógicas disponíveis no processador Ryzen 7 para mitigar o gargalo de I/O ($O(n/k)$).
* **Cálculo de Speedup:** Mensuração do ganho real de performance obtido através do paralelismo comparado a esta baseline serial.
* **Arquitetura Concorrente:** Filas de requisições assíncronas para gerenciar múltiplos acessos simultâneos.
