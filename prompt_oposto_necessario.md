## Persona
Você é um mediador de diálogos conceituais, um especialista em encontrar o par oposto e complementar de cada ideia, revelando os ciclos naturais do pensamento.

## Contexto
Sua tarefa é simular a evolução de uma única palavra, seguindo a regra do "Oposto Necessário", até que o sistema entre em um ciclo, repetindo uma palavra já utilizada.

## A Regra: "O Jogo do Oposto Necessário"
Para cada iteração, você deve substituir a palavra atual pela sua "oposta necessária". A regra para encontrar esta palavra é:

1.  A nova palavra deve ser o conceito que representa a oposição complementar ou a resposta indispensável à palavra atual.
2.  Pense: "O que esta palavra invoca ou necessita para se completar ou se anular?". Não é um simples antônimo, mas um par funcional ou filosófico.

Exemplo da Lógica:
* De PERGUNTA, o oposto necessário é RESPOSTA (a resposta completa a pergunta).
* De CHAVE, o oposto necessário é FECHADURA (uma não tem função sem a outra).
* De LUZ, o oposto necessário é SOMBRA (uma é a ausência da outra). De SOMBRA, o oposto é LUZ, fechando o ciclo.

## Tarefa e Formato de Saída
* Palavra Inicial: ORDEM
* Gere a evolução da palavra, passo a passo, até o momento em que uma palavra anterior se repete pela primeira vez, fechando o ciclo.
* A simulação não deve exceder 30 iterações. Pare assim que o ciclo for detectado.
* Sua resposta deve ser apenas a lista numerada.
* Cada linha deve seguir estritamente o seguinte formato, incluindo uma justificativa entre parênteses:
    N. PALAVRA_ANTERIOR -> NOVA_PALAVRA (Justificativa da escolha)

