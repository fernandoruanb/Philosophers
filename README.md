<h1 style="text-align: center;">Philosophers</h1>
<img src="https://cdn.pixabay.com/photo/2016/04/01/23/35/art-1301872_1280.png"\>

# About the Project

O projeto conhecido como **philosophers** propõe um desafio interessante e que já foi muito debatido desde 1965. Trata-se que termos uma quantidade **X** de filósofos e também uma quantidade de garfos, sendo um garfo por filósofo. Para que o mesmo possa se alimentar, ele precisa ter **dois garfos**, não conseguindo se alimentar se tiver sem nenhum ou apenas um único garfo. Logo, existem casos em que o filósofo terá que morrer definitivamente como se tivermos apenas um único filósofo, após passar o tempo de vida determinado, ele morrerá imediatamente.

Como funciona a lógica do projeto? Ao rodarmos o programa, precisamos passar alguns argumentos que são eles a **quantidade de filosófos, tempo de morte, tempo para se alimentar e tempo para dormir**, esses são os argumentos obrigatórios. Podemos colocar um argumento opcional que é sobre o **número de refeições** que cada filósofo tem que comer, caso todos tenham se alimentado na quantidade informada de vezes, o programa será encerrado automaticamente.

Durante a execução do programa, cada filósofo será representado por uma **thread**. Mas, o que é uma **thread**? Quando executávamos um programa, usávamos apenas uma única **thread**, pensemos nela como um operário único que, sozinho, executava todas as tarefas. Em programas que dizemos serem **multithreads** significa que existem vários funcionários trabalhando em parceria para a solução de um problema, certo? Em teoria, isso é algo muito bonito por conta que se temos um restaurante, é muito mais fácil dividir tarefas para termos um garçom, um cozinheiro, um faxineiro, um entregador e etc. O problema? Quando falamos de sistemas computacionais, se o "cozinheiro", por exemplo, precisar pegar um "garfo", acessar uma variável, e outro "cozinheiro" também precisar dessa variável, uma condição de disputa **data race** pode ocorrer. Dois cozinheiros disputando um garfo pode não ser considerado um problema tão grave, então, pensemos no caso de dois vigias, um vigia olha e não ver ameaça, seta uma variável como `0` para indicar **ausência de ameaça** e o outro vigia observa uma ameaça e vai lá e troca para `1` indicando um estado de **ameaça detectada** bem na hora que o outro vigia usa para dizer que não existe ameaça nenhuma. Ou seja, um evento importante foi soobrescrito e agora fica como se nunca tivesse ocorrido qualquer ameaça, uma terrível falta.

## How to solve

Para resolver o problema clássico de filósofos, precisaremos "dá ordem ao caos" introduzindo um controle nas threads, nossos funcionários, ao qual usamos as **mutexes**. Como elas funcionam, se um filósofo vai printar uma mensagem, o gerenciamento de buffer de um printf pode ser alvo dessa concorrência, então, um funcionário pega a chave, entra, os outros não conseguem entrar até ele devolver a chave, faz o serviço e depois ele libera a chave para que os demais possam continuar seus trabalhos. O que isso indica, se meu vigia (monitor) perceber que alguém morreu, ele liga a "flag da morte" como positiva e usa uma **mutex global**, quando acionada, todos param suas atividades, após isso voltam e conseguem pegar o estado atualizado. E se um filósofo precisar alterar os seus próprios dados, nesse caso, ele usa uma **mutex própria** dele, sua própria chave, não impossibilitando os outros de agirem, apenas para proteger a si mesmo. 

## How to verify

Durante vários projetos da 42, muitos cadetes, como chamamos os estudantes da famosa escola, usam ferramentas como o **valgrind** ou **fsanitize** para verificar leaks de memória, file descriptors, acessos inválidos em seus programas. No caso de programas multithread, usaremos uma ferramenta do valgrind feita para análise da concorrência de threads. Qual é ela? O famoso **helgrind**. Diferentemente do valgrind, ele analisa se existe possíveis competições por recursos acontecendo no seu programa e ele foi o nosso principal indicador para saber se tivemos ou não sucesso na resolução do problema clássico do jantar de filósofos.

A rotina do philosopher se resume em:

* pensar (thinking)
* pegar garfos (take a fork)
* Comer (eating)
* soltar os garfos (left the fork)
* dormir (sleeping)

E retorna o ciclo novamente.

O ciclo termina em caso de um filósofo morrer ou todos concluírem o total de refeições informado (argumento opcional). Como isso acontece? O primeiro argumento é o número de filósofos que irá representar o número de threads, o segundo é o tempo de vida (tudo em milisegundos), tempo para comer, tempo para dormir, número de refeições (opcional).

O filósofo pensa enquanto tenta pegar os garfos, quando ele consegue pegar dois garfos, ele começa a comer (consome o tempo definido para terminar a refeição), após isso ele solta os garfos e vai dormir (tempo de dormir é consumido). Primeiro problema aqui, se todos os filósofos forem pegar os garfos no mesmo sentido, como sempre o da esquerda ou direita, causaremos um problema chamado de **deadlock**, a fechadura da morte, onde nenhum deles conseguirá pegar todos os garfos e todos estarão condenados a morte iminente. Para evitar esse problema inicial clássico, os filósofos podem pegar dos garfos direito e esquerdo, alternando ou podemos nos basear em ímpar e par para definir uma ordem (filósofo ímpar sempre esquerdo, por exemplo) e o inverso no outro. 

Com isso, o filósofo pode morrer enquanto estiver comendo ou enquanto estiver dormindo. Por isso, temos um contador de alta precisão que conta para perceber quando o filósofo morreu enquanto estava nesses estados visto que temos uma tolerância máxima de até 10 milissegundos de atraso nessa detecção.

## How to execute

Para executar o programa do **Philosophers**, primeiro precisamos compilar o programa com esse comando:
```bash
make
```
Com o programa executado, podemos começar nossos testes facilmente usando o shell de nossa preferência:

```bash
./philo 2 300 200 100
```
Nesse caso acima, como exemplo, irá ter a seguinte saída, sujeito a mudanças, mas não ao resultado:

```bash
0 Philo 1 is thinking
0 Philo 1 has taken a fork
0 Philo 1 has taken a fork
0 Philo 1 is eating
0 Philo 2 is thinking
200 Philo 1 is sleeping
200 Philo 2 has taken a fork
200 Philo 2 has taken a fork
200 Philo 2 is eating
300 Philo 1 died
```

Percebam que o filósofo 1 pegou os dois garfos e começou a comer (consumirá 200 ms de tempo para isso). Quando ele foi dormir, seu tempo de vida se recupera quando ele começou a refeição, consumindo do sono esse novo tempo de vida, reduzindo em 100ms depois para a nova rodada, Quando o `Philo 2` começa a se alimentar, ele irá consumir **200 ms** de tempo sendo que já foi consumido **100 ms** no sono do `Philo 1` que tem **300 ms** de vida. O que isso significa? **Philo 1, deverá morrer** em 300ms. Como? O tempo dele foi resetado no eating, após ele comer, foi dormir, 100 ms foram consumidos, com os 200 ms do Philo 2 somam-se 300 ms que fazem ele morrer em 300ms.

```bash
./philo 1 800 200 200
0 Philo 1 is thinking
0 Philo 1 has taken a fork
800 Philosopher 1 died
```

Esse exemplo é de uma função personalizada apenas para esse caso. Ela indica que o `Philosopher 1` morreu após aguardar o tempo esperado já que é um caso onde ele morrerá de forma iminente por nunca conseguir um segundo garfo.

```bash
./philo 2 200 200 200
0 Philo 2 is thinking
0 Philo 2 has taken a fork
0 Philo 2 has taken a fork
0 Philo 2 is eating
0 Philo 1 is thinking
200 Philo 1 died
```
Nesse exemplo, assim que o `Philo 2` começar a comer, seu tempo é resetado, mas, o `Philo 1` possui seu tempo de morte para 200ms, ou seja, quando o `Philo 2` parar de comer, o `Philo 1` deverá morrer.

