# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
O printf() é uma função realiza bufferização da saída antes mesmo de chamar write(), podendo haver mais ou menos chamadas para o write() dependendo do tamanho da mensagem e do momento em que o buffer é descarregado. Ao mesmo tempo, o write() é uma chamada de função no kernel direta, na qual cada invocação no código gera exatamente uma syscall correspondente.
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
O write() é mais previsível, pois sua uma correspondência é direta com cada chamada na aplicação e a syscall executada, enquanto o printf(), ao usar buffer interno, pode gerar resultados diferentes em dependendo das situações distintas.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
O file descriptor usado foi o 3, pois os descritores 0, 1 e 2 já estavam reservados pelo sistema para a entrada padrão, sendo stdin = 0, saída padrão, sendo stdout = 1, e saída de erro padrão com stderr = 2.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
No strace, a chamada read() retornou 127, que foi o número de bytes passado no BUFFER_SIZE - 1.

```

**3. Por que verificar retorno de cada syscall?**

```
Verifica-se o retorno pois cada syscall pode falhar, e ao fazer isso, o programa pode detectar e tratar esses erros em vez de só continuar executando. Além disso, elas são muito importantes na programação de baixo nível, já que as syscalls não lançam exceções e apenas retornam valores que indicam sucesso ou falha.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 130
- Tempo: 0.002654 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          | 130             | 0.002355  |
| 64          | 130             |  0.002654 |
| 256         |   130           |0.002664   |
| 1024        |  130            | 0.002640  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o tamanho do buffer, menos syscalls são feitas, uma vez que mais dados são lidos e escritos de uma vez só, melhorando o desempenho e reduzindo o tempo gasto com I/O.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Nem sempre o read() retorna exatamente o tamanho do buffer. Ele tenta ler até esse valor, mas pode vir menos, dependendo do que tem disponível. Se estiver no final do arquivo, ele só vai trazer o que sobrou, e por isso, é importante conferir a quantidade de bytes que foram realmente lidos.
```

**3. Qual é a relação entre syscalls e performance?**

```
Syscalls têm relação direta com a performance porque cada vez que o programa precisa falar com o sistema operacional, ele faz uma pausa, troca de contexto, custando tempo. Quanto mais syscalls, mais lento pode ser. Por isso, é melhor fazer menos chamadas e aproveitar bem cada uma, como usando buffers maiores pra ler ou escrever mais dados de uma vez.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000214 segundos
- Throughput: 6224.45 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ ] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Verifica-se bytes_escritos == bytes_lidos pra garantir que tudo o que foi lido do arquivo de origem foi realmente escrito no arquivo de destino, não entregando assim um arquivo incompleto ou corrompido.
```

**2. Que flags são essenciais no open() do destino?**

```
A flag O_WRONLY, que abre o arquivo para escrita apenas, O_CREAT, que cria um arquivo se ele não existir, e o O_TRUN, que apaga o conteúdo anterior e começa do zero o arquivo.
```

**3. O número de reads e writes é igual? Por quê?**

```
O número de chamadas read() e write() geralmente é igual porque, cada vez que o programa lê um pedaço do arquivo de origem, ele escreve esse mesmo pedaço no arquivo de destino.
```

**4. Como você saberia se o disco ficou cheio?**

```
Se o disco ficar cheio, o programa não consegue mais escrever no arquivo. A função write() daria erro e retornaria -1, mostrando que não tem mais espaço disponível.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Se a gente esquece de fechar os arquivos, o sistema pode ficar com eles abertos na memória, o que ocupa recursos e pode causar problemas. Além disso, pode acontecer de os dados não serem salvos direitinho no arquivo, porque o fechamento garante que tudo foi escrito e finalizado. 
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls são a forma de um programa em modo usuário pedir serviços ao kernel.
Quando chamamos  os comendos (read, write, open...) o programa passa do espaço do usuário para o kernel, o SO executa a ação e depois retorna o resultado ao usuário.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os file descriptors são importantes porque funcionam como identificadores numéricos que o sistema operacional usa para controlar entradas e saídas, e arquivos. Eles permitem que o programa se refira a arquivos, sockets ou dispositivos de forma unificada, sem precisar saber detalhes do hardware.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
O tamanho do buffer tem relação direta com a performance porque ele define o quanto de dados o programa lê ou escreve de uma vez. Se o buffer for pequeno, o programa precisa fazer várias chamadas de leitura e escrita, o que deixa tudo mais lento. Já com um buffer maior, ele consegue transferir mais dados por vez, fazendo menos chamadas e ganhando velocidade.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** o time ./ex4_copia foi mais rápido 
(real    0m0.003s
user    0m0.001s
sys     0m0.002s)


**Por que você acha que foi mais rápido?**

```
O time ./ex4_copia foi mais rápido pois o time cp dados/origem.txt dados/destino_cp.txt usa o comando cp do sistema, que é mais completo e faz várias verificações a mais / extras, como permissões, atributos e metadados.

```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
