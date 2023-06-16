## Autores

Júlia Evelyn de Oliveira Silva
Letícia de Assis Fraga
Matheus Nolasco Miranda Soares
Pedro Henrique Rodrigues Ferreira
Vinícius Levi Viana de Oliveira

## Introdução

Criado em 2015, o Discord é uma plataforma web e mobile focada em comunicação de grupos, que possui uma arquitetura distribuída e escalável. Algumas de suas features principais são as chamadas de voz, assim como o envio de mensagens. Em 2020, o sistema possuía mais de 300 milhões de usuários registrados. Todos os dias, são gastos o equivalente a 4 bilhões de minutos com envio de mensagens de texto e chamadas de voz ou de vídeo.

Nesse site você conhecerá um pouco mais sobre a arquitetura do sistema assim como detalhes sobre a mesma, divirta-se!

Nota: Aqui, nós utilizaremos o termo "guilda" (Guild, em inglês) para referir ao conjunto de usuários, que são chamados de "servidores" no cliente da aplicação. Portanto, "servidores" nesse site se referem ao conceito de engenharia de software.

## Visão geral

Existem 3 serviços principais na arquitetura do Discord: Discord Gateway, Discord Guilds e Discord Voice.

- Discord Gateway: Permite que os clientes abram conexões seguras de WebSocket com o Discord. Gerencia a autenticação, a comunicação em tempo real e o envio de mensagens entre os clientes e os serviços do Discord.
- Discord Guilds: Gerenciam informações relacionadas às guildas, como membros, canais e permissões de usuários. Cada guilda possui seus próprios canais de texto e voz, membros e configurações personalizadas.
- Discord Voice: Permite a comunicação de voz em tempo real. Gerencia a codificação e decodificação de áudio, a sincronização de voz entre os participantes e o roteamento do áudio entre os usuários.

## Performance

O Discord mantém a performance de sua aplicação mesmo após a evolução de seu software. Mas como isso é feito? A equipe de desenvolvimento utiliza uma abordagem iterativa e focada na melhoria contínua.

Uma das estratégias adotadas para manter a performance é a otimização do código e dos algoritmos utilizados. Eles procuram constantemente maneiras de tornar o software mais eficiente e rápido, reduzindo o tempo de carregamento e resposta em diferentes áreas da plataforma.

A equipe de desenvolvimento também realiza monitoramento constante do desempenho da aplicação, utilizando métricas e ferramentas de análise para identificar possíveis gargalos e problemas de desempenho. Dessa forma, eles podem agir rapidamente para corrigir essas questões e melhorar a experiência do usuário.

No geral, o Discord adota uma abordagem abrangente para garantir a performance de sua aplicação, combinando otimizações técnicas, arquitetura escalável, monitoramento constante e testes rigorosos. Isso permite que os usuários desfrutem de uma experiência suave e responsiva ao utilizar a plataforma.

## Escalabilidade

O discord foi criado, desde seu protótipo, em [Elixir](https://elixir-lang.org/), uma linguagem de programação funcional que é executada na máquina virtual “Erlang VM”. O uso dessa linguagem foi fundamental para permitir a alta escalabilidade do Discord, já que ela é uma linguagem moderna, amigável e desenvolvida para aplicações que visam alta escalabilidade, por possibilitar a execução de múltiplos processos simultaneamente sem comprometer a perfomance da aplicação. Além disso, ela fornece para o desenvolvedor um ambiente bem destribuído que facilita muito a escalabilidade horizontal, ideal para sistemas de rápido crescimento. 

Como o Discord é um aplicativo de comunicação entre usuários, a velocidade é um requisito fundamental, e essa também foi a maior barreira que os desenvolvedores enfrentaram para alcançar a alta escalabilidade desejada. Com o uso do Elixir, foi possível dividir os processos de envio de mensagens em diversas fases diferentes, ao invés de apenas uma, e com isso, foi possível reduzir pela metade o tempo de resposta que tinham. Com isso eles conseguiram escalar facilmente enquanto mantiveram a excelência do sistema.

Em relação aos serviços disponibilizados para os usuários, mais de 850 servidores para esses serviços são executados simultaneamente em 13 regiões, hospedados em mais de 30 centros de dados ao redor do mundo. Com essa arquitetura, o Discord pode atender mais de 2,6 milhões de usuários de voz simultâneos, com tráfego de saída de mais de 220 Gbps (bits por segundo) e 120 Mpps (pacotes por segundo).

## Tolerância a falhas

Com a disponibilização de 850 servidores distribuídos geograficamente mencionada anteriormente, o sistema fornece redundância para lidar com falhas e ataques DDoS a servidores (ateques que sobrecarregam a aplicação para deixar o serviço indisponível). Além disso, o a distribuição permite que os clientes tenham acesso aos serviços com uma latência menor.

## Referências

_Discord Blog. Disponível em: https://discord.com/blog/. Acesso em maio de 2023_

_BROWN, Abram. Disponível em: https://www.forbes.com/sites/abrambrown/2020/06/30/discord-was-once-the-alt-rights-favorite-chat-app-now-its-gone-mainstream-and-scored-a-new-35-billion-valuation/. Acesso em maio de 2023_

