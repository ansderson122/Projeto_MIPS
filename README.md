# Descrição

Este projeto consistiu na criação de um processador capaz de calcular a multiplicação de matrizes 4x4. O processador foi desenvolvido com base na arquitetura MIPS.

## Comandos

Os seguintes comandos foram implementados:

| Comandos | Binário | Hexadecimal |
|----------|---------|-------------|
| add      | 1001    | 9           |
| sub      | 1011    | b           |
| mul      | 0100    | 4           |
| li       | 0100    | 4           |
| lw       | 1100    | c           |
| sw       | 0011    | 3           |
| j        | 0001    | 1           |
| bge      | 0010    | 2           |

1. `add` - Soma dois registradores. Por exemplo, em "9f810000" soma o registro 8 com 1 e salva em f.
2. `sub` - Subtrai dois registradores, com estrutura de comando semelhante ao add.
3. ` mul`  - Multiplica dois registradores, com estrutura de comando semelhante ao add.
4. ` li`  - Carrega um imediato no registrador. Por exemplo, "44100000" carrega o 16 no registrador 4. Este comando usa o 3º e 4º bit para o número a ser carregado, e 10 em hexadecimal é 16 em decimal.
5. ` lw` - Lê o conteúdo da memória e salva em um registrador. Por exemplo, "c4050000" carrega no registrador 4 o conteúdo contido na memória no endereço contido no registrador 5. O 3º bit tem que ser 0 para o comando funcionar.
6. ` sw`  - Escreve na memória o conteúdo do registrador em um endereço. Por exemplo, "30510000" escreve na memória o conteúdo do registrador 5 no endereço contido no registrador 1. O 2º bit tem que ser zero.
7. ` j`  - Salta para o endereço na memória de instrução. Por exemplo, "11000000" salta para a instrução 16.
8. ` bge`  - Compara os dois registradores e, se o primeiro for maior ou igual, salta para o endereço da memória de instrução. Por exemplo, "20628800", compara se o registrador 6 é maior ou igual ao registrador 8, se for verdadeiro, salta para o endereço 88 da memória de instrução. O 2º bit tem que ser 0, e o 5º e o 6º são o endereço.


# Implementação do algoritmo para multiplicar matrizes

| Instrução em Hexadecimal  | Instrução | Descrição                              |
|----------|-----------|----------------------------------------|
| 4101     | li r1,1    | Carrega o valor 1 no registrador r1   |
| 4204     | li r2,4    | Carrega o valor 4 no registrador r2   |
| 4300     | li r3,0    | Carrega o endereço da primeira matriz no registrador r3 |
| 4440     | li r4,64   | Carrega o endereço da segunda matriz no registrador r4 |
| 4e80     | li re,128  | Carrega o endereço onde o resultado será armazenado no registrador re |
| 4f10     | li r16,16  | Carrega o valor 16 no registrador r16 |
| 4500     | li r5,0    | Inicializa o contador i                |
| 2052ff      | bge $r5,4, end_i_loop | Se i >= 4, saia do loop |
| 4600     | li r6,0    | Inicializa o contador j                |
| 206288      | bge $r6,4, end_j_loop | Se j >= 4, saia do loop |
| 4700     | li r7,0    | Inicializa o contador k                |
| 4c00     | li r12,0   | Inicializa o acumulador                |
| 20726c      | bge $r7,4, end_k_loop | Se k >= 4, saia do loop |
|    **Loop para cálculo da matriz resultante:** |
| ba5f     | mul $r10, $r5, $r16 | Calcula o endereço da linha i na primeira matriz |
| bd72     | mul $r13, $r7, $r2  | Calcula o endereço da coluna k na segunda matriz |
| 9aad     | add $r10, $r10, $r13 | Adiciona os endereços calculados |
| 9aa3     | add $r10, $r10, $r3  | Adiciona o endereço da primeira matriz |
| c80a     | lw $r8,0($r10)       | Carrega o valor da primeira matriz |
|      **Cálculo do endereço e carregamento da segunda matriz:** |
| ba7f     | mul $r10, $r7, $r16 | Calcula o endereço da linha k na segunda matriz |
| bd62     | mul $r13, $r6, $r2  | Calcula o endereço da coluna j na segunda matriz |
| 9aad     | add $r10, $r10, $r13 | Adiciona os endereços calculados |
| 9aa4     | add $r10, $r10, $r4  | Adiciona o endereço da segunda matriz |
| c90a     | lw $r8,0($r10)       | Carrega o valor da segunda matriz |
|       **Cálculo da multiplicação e acumulação:** |
| bd89     | mul $r13, $r8, $r9  | Multiplica os valores das matrizes |
| 9ccd     | add $r12, $r12, $r13| Acumula o resultado da multiplicação |
| 9771     | add $r7, $r7, $1    | Incrementa o contador k               |
| 1300        |  | Volta para o início do loop de k |
|        **Armazenamento do resultado na matriz final:** |
| ba5f     | mul $r10, $r5, $r16 | Calcula o endereço da linha i na matriz resultante |
| bd62     | mul $r13, $r6, $r2  | Calcula o endereço da coluna j na matriz resultante |
| 9aad     | add $r10, $r10, $r13| Adiciona os endereços calculados |
| 9aae     | add $r10, $r10, $re | Adiciona o endereço onde o resultado será armazenado |
| 30ca     | sw $r12, $r10       | Armazena o resultado na matriz final |
| 9661     | add $r6, $r6, $r1   | Incrementa o contador j               |
| 1240        |  | Volta para o início do loop de j |
|   **Incremento do contador i e volta ao início:** |
| 9551     | add $r5, $r5, $r1   | Incrementa o contador i               |
| 11c0 |  | Volta para o início do loop de i |



# Implementação do processador

## Unidade de controle

A tabela abaixo relaciona os comandos com as saídas da Unidade de Controle:

| Comandos | ula_op | write_reg | read_mem | write_mem | saltar | li |
|----------|--------|-----------|----------|-----------|--------|----|
| add      | 1001   | 1         | 0        | 0         | 0      | 0  |
| sub      | 1011   | 1         | 0        | 0         | 0      | 0  |
| mul      | 0100   | 1         | 0        | 0         | 0      | 0  |
| li       | 0100   | 1         | 0        | 0         | 0      | 1  |
| lw       | 1100   | 1         | 1        | 0         | 0      | 0  |
| sw       | 0011   | 0         | 0        | 1         | 0      | 0  |
| j        | 0001   | 0         | 0        | 0         | 1      | 0  |
| bge      | 0010   | 0         | 0        | 0         | 0      | 0  |

1. `ula_op` indica para a ULA qual operação deve ser realizada.
2. `write_reg` indica que a operação pode escrever no banco de registradores.
3. `read_mem` indica que a operação pode ler a memória de dados.
4. `write_mem` indica que a operação pode escrever na memória de dados.
5. `saltar` indica que haverá um salto.
6. `li` indica para salvar o valor diretamente no banco de registradores.

![](./img/UC.png)

# Processado 
![](./img/PRO.jpg)

