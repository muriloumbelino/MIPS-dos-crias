# Relatório Técnico -  MIPS GLMV

## Introdução
O presente relatório documenta a concepção, implementação e o funcionamento de um processador customizado baseado na arquitetura MIPS, desenvolvido integralmente no ambiente de simulação Logisim. O objetivo central deste projeto é consolidar os conceitos teóricos de Arquitetura de Computadores através da construção prática de um caminho de dados (*Datapath*) funcional e de sua respectiva Unidade de Controle.
Para fins didáticos e visando a otimização da legibilidade do circuito, o processador foi projetado com um barramento de dados de 8 bits e um formato de instrução de 18 bits, adotando a Arquitetura Harvard para a separação física entre as memórias de dados e instruções. Além de focar em um roteamento visualmente limpo com o uso de túneis lógicos, o projeto destaca-se pela implementação de uma Unidade Lógica e Aritmética (ULA) expandida, capaz de processar operações complexas (como multiplicação e divisão) de forma nativa no hardware. 
Este documento detalha o diagrama de blocos arquitetural, os submódulos que compõem o sistema e as justificativas técnicas que guiaram as decisões de engenharia da equipe durante o desenvolvimento.

## Diagrama de Blocos

<img width="1044" height="601" alt="image" src="https://github.com/user-attachments/assets/64a1dc70-f7fd-481a-b803-599b627afff1" />


## Descrição Detalhada do Papel e Funcionamento de Cada Módulo

### 1. Program Counter (PC)
* **Papel:** Atua como o registrador de estado que armazena o endereço da próxima instrução a ser processada.  
* **Funcionamento:** Em condições normais, ele incrementa o endereço sequencialmente. No entanto, ele possui lógica para realizar desvios, recebendo sinais de `Jump` e `Branch` da Unidade de Controle e o sinal `Equal` da ULA para decidir se o próximo endereço será o sucessor imediato ou um destino de salto.  

### 2. Memória de Instruções (ROM)
* **Papel:** Armazena o código binário do programa (instruções de 18 bits) que define o comportamento do sistema.  
* **Funcionamento:** Recebe o endereço vindo do PC e disponibiliza a instrução completa no barramento. Ela decompõe essa instrução para enviar o `Opcode` para a UC, os campos de registradores (`rs`, `rt`, `rd`) para o Banco de Registradores, o valor de deslocamento (`shamt`) para a ULA e o código de função (`funct`) para o Controlador da ULA.  

### 3. Banco de Registradores
* **Papel:** Fornece armazenamento rápido para operandos e resultados intermediários.  
* **Funcionamento:** Contém 8 registradores de 8 bits. Utiliza sinais de controle `RegDst` para selecionar o registrador de destino e `RegWrite` para autorizar a escrita. O módulo permite a leitura simultânea de dois valores (`DataRs` e `DataRt`) com base nos índices fornecidos pela ROM.  

### 4. Unidade de Controle (UC)
* **Papel:** Coordena a ativação de todos os componentes do processador através da decodificação da instrução.  
* **Funcionamento:** Implementada com lógica combinacional, recebe o `Opcode` da ROM e gera os sinais de controle necessários para o fluxo de dados, incluindo permissões de memória (`MemRead`/`MemWrite`), seleção de escrita em registradores e o tipo de operação que a ULA deve preparar (`ALUOp`).  

### 5. Operador da ULA (Controlador da ULA)
* **Papel:** Refina o comando da UC para a operação específica que a ULA deve realizar.  
* **Funcionamento:** Recebe o sinal genérico `ALUOp` da Unidade de Controle e o campo `funct` da instrução ROM para produzir o sinal de função específico que seleciona a operação aritmética ou lógica final.  

### 6. ULA (Unidade Lógica e Aritmética)
* **Papel:** Executa o processamento de dados propriamente dito.  
* **Funcionamento:** Recebe os dados do Banco de Registradores e a função do Operador da ULA. Ela realiza operações de soma, subtração, multiplicação, divisão, negação e deslocamento. Além do resultado (`ResULA`), ela gera o sinal `Equal` comparando os operandos para auxiliar em instruções de desvio condicional.  

### 7. Memória de Dados (RAM)
* **Papel:** Armazena dados de forma volátil durante a execução, permitindo persistência além dos registradores.  
* **Funcionamento:** Utiliza o resultado da ULA como endereço para as operações. Ela lê ou escreve o conteúdo do registrador `DataRt` dependendo dos sinais `MemRead` e `MemWrite` enviados pela UC. O resultado da leitura (`WriteData`) é enviado de volta para ser armazenado no Banco de Registradores via sinal `MemToReg`.


## Relato das Decisões de Projeto e Justificativas Técnicas

1. **Base Referencial e Evolução do Projeto:**
   O desenvolvimento estrutural do processador teve como ponto de partida metodológico a série em vídeo do professor Bruno Abreu. A partir dessa fundamentação teórica, a equipe projetou modificações arquiteturais e visuais substanciais para adequar o circuito aos critérios de legibilidade, organização e eficiência exigidos na avaliação.

2. **Correção Lógica e Visual do Sinal `MemToReg`:**
   Durante a fase de implementação, a equipe identificou uma inconsistência na arquitetura base do vídeo em relação à lógica de saída da RAM. No projeto original, o dado da memória era enviado diretamente para o registrador quando o circuito de controle (`MemToReg`) estava desativado (em nível lógico baixo). Para corrigir essa semântica e garantir que a transferência de dados exigisse a ativação explícita do circuito, a lógica desse sinal foi barrada (invertida) internamente na Unidade de Controle (UC) e as entradas do multiplexador (MUX) na saída da RAM foram invertidas. Essa adaptação técnica tornou o acionamento do sinal coerente com a ação executada e deixou o *Datapath* mais intuitivo.

3. **Uso Estratégico de Túneis (*Tunnels*):**
   Em substituição ao roteamento físico contínuo (fios cruzando todo o projeto), optou-se pela utilização de túneis lógicos para a distribuição dos barramentos de dados e dos sinais da Unidade de Controle. Essa decisão técnica visa maximizar a legibilidade do circuito no Logisim, encapsulando a complexidade e permitindo a rápida identificação de falhas e o acompanhamento claro da execução das instruções.


## Conclusão

A implementação do processador MIPS 8-bits no Logisim atendeu com êxito a todos os requisitos de funcionamento propostos. O desenvolvimento prático permitiu uma compreensão profunda da interação entre o fluxo de dados e os sinais emitidos pela Unidade de Controle. Decisões de projeto, como a correção da lógica de acionamento do sinal `MemToReg` e a utilização de túneis, garantiram não apenas o rigor técnico do sistema, mas também uma topologia de circuito organizada, profissional e de fácil depuração visual.
A validação do hardware através de algoritmos reais, como o cálculo da Sequência de Fibonacci, confirmou a robustez da ULA customizada.
