# 👥 Plataforma Distribuída para Busca de Pessoas Desaparecidas

Alunos:
Pauline Fernandes Braz / 076961
Nathan David Oliveira Fernandes / 077992

## 📋 Sobre o Projeto
Este sistema foi desenvolvido como parte prática da disciplina de **Sistemas Distribuídos e Concorrentes**. O objetivo principal é simular uma plataforma de segurança pública de alta escala: o sistema recebe fotos de suspeitos enviadas em tempo real por cidadãos via aplicativo (Concorrência), processa e compara essas imagens utilizando múltiplos núcleos da CPU (Paralelismo) e distribui a carga de busca entre servidores regionais (Sistemas Distribuídos).

---

## 💾 Engenharia de Dados e Volumetria (2 GB)
Para validar o sistema em um cenário severo de alta volumetria, foi utilizado o dataset público **CelebA**:
* **Massa de Dados:** 202.599 imagens reais de rostos (aproximadamente 2 GB em disco).
* **Link Oficial do Dataset:** [Kaggle - CelebA Dataset](https://www.kaggle.com/datasets/jessicali9530/celeba-dataset)
* **Arquitetura de Armazenamento:** Em sistemas de produção de larga escala, imagens brutas nunca são guardadas diretamente dentro de tabelas de bancos de dados relacionais para evitar estouro de memória e lentidão de I/O. A arquitetura deste projeto segue o padrão de mercado:
  1. As imagens físicas ficam armazenadas em um *Storage* (diretório local de arquivos).
  2. O banco de dados em memória RAM gerencia apenas os metadados (nome/caminho do arquivo) e o seu respectivo **Vetor de Características (Embedding)** de 128 posições gerado por processamento matemático.

---

## 🔬 Técnica de Processamento e Reconhecimento Facial
O sistema utiliza processamento de matrizes numéricas através da biblioteca `OpenCV` associada à extração vetorial com suporte da biblioteca `NumPy`. 
1. **Vetorização:** As imagens de cadastro e do suspeito são convertidas para escala de cinza e redimensionadas para uma matriz matemática estável, gerando um vetor unidimensional de características numéricas normalizadas entre `0.0` e `1.0`.
2. **Comparação:** A busca e validação de identidade ocorrem através do cálculo da **Distância Euclidiana** entre o vetor do suspeito e os vetores do banco:

$$d = \sqrt{\sum_{i=1}^{n} (q_i - p_i)^2}$$

Onde $q$ é o vetor do suspeito capturado na rua e $p$ é o vetor do registro do banco. Uma distância estatística menor que `0.6` estabelece uma correspondência positiva de identidade (Match).

---

## 🧪 Cenário de Teste Atual: Varredura Serial (Baseline)
Nesta primeira etapa do projeto, o algoritmo foi executado em modo **Serial Monothread** (utilizando estritamente 1 único núcleo do processador sem concorrência) para servir como *Baseline* (linha de base de performance) para as futuras otimizações concorrentes.

### Configuração do Pior Cenário ($O(n)$):
Para simular o cenário mais crítico do algoritmo — onde o sistema precisa varrer a base de dados inteira antes de encontrar o alvo ou para confirmar um alarme falso —, a foto de cadastro do alvo (`nathan_cadastro.jpg`) foi isolada e posicionada **estritamente na última posição** da lista de mais de 202 mil registros. Uma imagem secundária (`nathan_suspeito.jpg`) foi submetida como a requisição de busca do aplicativo.

### ⏱️ Resultados Obtidos no Ambiente Local

| Métrica Avaliada | Resultado Registrado |
| :--- | :--- |
| **Total de Registros de Fundo (CelebA)** | 202.599 fotos |
| **Alvo Inserido no Banco** | 1 foto (`nathan_cadastro.jpg`) |
| **Volume Total da Base de Busca** | 202.600 registros |
| **Posição de Localização do Alvo** | Índice 202.599 (Último elemento) |
| **Status do Reconhecimento Facial** | **🎯 SUCESSO (Alvo Localizado)** |
| **Tempo Total de Varredura Serial** | **0.4252 segundos** |

---

## 🚀 Próximos Passos do Desenvolvimento Concorrente
O tempo de **0.4252 segundos**, embora seja veloz para atender a um único usuário na arquitetura de memória RAM, torna-se um gargalo crítico em sistemas distribuídos sob regime de alta concorrência (ex: milhares de requisições simultâneas vindas de aplicativos móveis da polícia na rua). 

Na próxima fase do projeto, implementaremos:
* **Paralelismo de Dados (Multi-threading):** Divisão da base de 202.600 vetores em fatias iguais (*chunks*) para serem processadas simultaneamente pelos múltiplos núcleos físicos da CPU ($O(n/k)$).
* **Arquitetura Concorrente:** Filas de requisições assíncronas para evitar o travamento ou enfileiramento do servidor principal.
