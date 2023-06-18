## Autores

Júlia Evelyn de Oliveira Silva <br/>
Letícia de Assis Fraga <br/>
Fraga Matheus Nolasco Miranda Soares <br/>
Pedro Henrique Rodrigues Ferreira <br/>
Vinícius Levi Viana de Oliveira <br/>

## Introdução

Criado em 2015, o Discord é uma plataforma web e mobile focada em comunicação de grupos, que possui uma arquitetura distribuída e escalável. Algumas de suas features principais são as chamadas de voz, assim como o envio de mensagens. Em 2020, o sistema possuía mais de 300 milhões de usuários registrados. Todos os dias, são gastos o equivalente a 4 bilhões de minutos com envio de mensagens de texto e chamadas de voz ou de vídeo.

Nesse site você conhecerá um pouco mais sobre a arquitetura do sistema assim como detalhes sobre a mesma, divirta-se!

Nota: Aqui, nós utilizaremos o termo "guilda" (Guild, em inglês) para referir ao conjunto de usuários, que são chamados de "servidores" no cliente da aplicação. Portanto, "servidores" nesse site se referem ao conceito de engenharia de software.

## Visão geral

Existem 3 serviços principais na arquitetura do Discord: Discord Gateway, Discord Guilds e Discord Voice.

- Discord Gateway: Permite que os clientes abram conexões seguras de WebSocket com o Discord. Gerencia a autenticação, a comunicação em tempo real e o envio de mensagens entre os clientes e os serviços do Discord.
- Discord Guilds: Gerenciam informações relacionadas às guildas, como membros, canais e permissões de usuários. Cada guilda possui seus próprios canais de texto e voz, membros e configurações personalizadas.
- Discord Voice: Permite a comunicação de voz em tempo real. Gerencia a codificação e decodificação de áudio, a sincronização de voz entre os participantes e o roteamento do áudio entre os usuários.

Quando um usuário está online, o cliente mantém uma conexão WebSocket com a Discord Gateway. O cliente recebe eventos relacionados a guildas, canais, mensagens, presença, etc., por meio dessa conexão.

Quando ele se conecta a um canal de voz, o status da conexão é representado pelo [objeto de state de voz](https://discord.com/developers/docs/resources/voice). O cliente atualiza o state de voz usando a conexão WebSocket do gateway e o usuário é atribuído a um servidor Discord Voice, que é responsável por transmitir o áudio de cada membro para o canal. Todos os canais de voz dentro de uma guilda são atribuídos ao mesmo servidor Discord Voice. Se o usuário é o primeiro participante de voz na guilda, o Discord Guilds é responsável por atribuir um servidor à guilda.

![Arquitetura geral](/general-architecture.png)

## Requisitos

Para atender os 3 serviços sem correr o risco da aplicação falhar em algum aspecto, foram elegidos alguns requisitos importantes para o funcionamento do Discord. São eles:

- Armazenamento
  - O usuário deve ser capaz de encontrar qualquer mensagem que já tenha sido enviada em suas conversas, independente da data de envio ser antiga ou não.
- Performance
  - O usuário deve ser capaz de manter uma conversa em uma chamada sem se preocupar com a quantidade de usuários que estão participando e outros fatores que possam prejudicar o funcionamento da plataforma e sua experiência, seja ela casual, gamer ou profissional.
- Escalabilidade
  - Os milhões de usuários devem ser capazes de realizar múltiplas tarefas simultaneamente ser se preocupar com a performance da plataforma.
- Tolerância a falhas
  - Os usuários devem utilizar a plataforma sem se preocupar com a segurança dos seus dados ou o perigo de ocorrer um vazamento das suas conversas e informações, além de poder utilizar sem se preocupar com bugs e outros impedimentos para sua experiência.

Nos tópicos abaixo, explicaremos melhor cada um dos requisitos, suas soluções e quais decisões do projeto levaram ao diagrama arquitetural final.

## Armazenamento

A versão inicial do discord foi feita em 2015 e desenvolvida em apenas dois meses. Desde o início o planejamento foi feito já pensando na evolução do software e deixando em aberto as possíveis soluções de escalonamento e melhorias futuras, assim como diz o lema da empresa: “build quickly to prove out a product feature, but always with a path to a more robust solution”, em português, construir rapidamente para lançar uma nova ferramenta ao produto, porém, sempre com um caminho para uma solução mais robusta.

Para armazenar as informações da aplicação inicial como as mensagens e tudo o que fosse necessário, eles utilizaram uma única instância do MongoDB, até a latência começar a incomodar e ser decidido que era hora de organizar melhor o banco de dados. Isso ocorreu logo em novembro do mesmo ano, quando eles alcançaram 100 milhões de mensagens armazenadas.

Deve-se lembrar que um dos requisitos do Discord é nunca apagar as mensagens enviadas em qualquer momento, para que os usuários sempre tenham acesso às mensagens antigas de uma conversa.

Como requisitos para a mudança de banco de dados e arquitetura, eles tinham como base a forma com as quais os usuários utilizavam a plataforma, e chegaram às seguintes conclusões:

- Tanto a escrita como a leitura eram relevantes para o sistema, pensando nos usuários que usam o chat privado e chegam a enviar e receber de 100 mil mensagens a 1 milhão em um dia facilmente.
- A busca não deveria ser mais custosa que o armazenamento em si.
- O armazenamento deveria suportar bilhões de mensagens.
- Não valeria a pena indexar mensagens de um chat caso os usuários não fizesse pelo menos uma busca de mensagens antigas

Como solução, eles escolheram utilizar o Cassandra, elasticsearch, e a modelagem de dados KKV, de forma que a chave primária seria composta por dois valores: channel_id e message_id.

Por fim e até os dias atuais, chegando em trilhões de mensagens a serem armazenadas, em 2020 chegaram à conclusão que o Cassandra já não atendia mais os seus requisitos. O Cassandra passou a ter problemas de performance, tornando a latência imprevisível para os usuários, além de apresentar manutenção custosa.

![Cassandra DB](/db-old.png)


Por isso, o Discord migrou todos os seus sistemas para ScyllaDB, mantendo sua modelagem anterior KKV. Para isso, outra decisão tomada foi a escolha de Rust para a escrita dos novos serviços de dados, que são intermediários entre a API e os clusters de ScyllaDB. O Rust, além de ser uma boa linguagem de programação em termos de segurança, bibliotecas e compilador que mostra mensagens claras, tem a concorrência como um de seus maiores benefícios, por isso se vários usuários estiverem solicitando a mesma informação do banco ao mesmo tempo, a consulta será feita apenas uma vez, como ilustrado na imagem abaixo.

O primeiro usuário que fizer uma solicitação fará com que uma task seja iniciada no serviço de dados, e as solicitações seguintes verificarão a existência dessa tarefa e se inscreverão nela. A task consultará o banco de dados e retornará a informação para todos os inscritos. Esse comportamento é interessante, por exemplo, em uma guilda grande que dispara uma mensagem para todos os usuários. Todos os usuários, assim que receberem a mensagem, podem ler, disparando vários requests ao mesmo tempo para o serviço de banco de dados. Nesse caso, concorrência reduz a quantidade de tráfego gerada no banco.

![Rust concurrency](https://assets-global.website-files.com/5f9072399b2640f14d6a2bf4/6406629e7ba3569d3c32c8ed_Example%201%402x.png)

Dessa forma, foi possível alcançar a evolução de 99% das pesquisas de mensagens antigas (read) que antes duravam entre 40ms e 125ms no Cassandra, agora duram 15ms de latência, e o armazenamento de mensagens no banco (write) que durava entre 5ms e 70ms, agora se mantém em 5ms.

![Migration](/db-migration.png)

## Performance

O Discord mantém a performance de sua aplicação mesmo após a evolução de seu software. Mas como isso é feito? A equipe de desenvolvimento utiliza uma abordagem iterativa e focada na melhoria contínua.

Uma das estratégias adotadas para manter a performance é a otimização do código e dos algoritmos utilizados. Os desenvolvedores procuram constantemente maneiras de tornar o software mais eficiente e rápido, reduzindo o tempo de carregamento e resposta em diferentes áreas da plataforma.

Eles utilizam uma metodologia chamada Code Splitting, que consiste em carregar o código conforme a demanda. Na maioria dos casos, ao carregar uma página, todo o código dela é executada no momento do acesso, porém isso diminui a performance do sistema a medida em que ele evolui.

A equipe de desenvolvimento também realiza monitoramento constante do desempenho da aplicação, utilizando métricas e ferramentas de análise para identificar possíveis gargalos e problemas de desempenho. Dessa forma, eles podem agir rapidamente para corrigir essas questões e melhorar a experiência do usuário.

No geral, o Discord adota uma abordagem abrangente para garantir a performance de sua aplicação, combinando otimizações técnicas, arquitetura escalável, monitoramento constante e testes rigorosos. Isso permite que os usuários desfrutem de uma experiência suave e responsiva ao utilizar a plataforma.

![Code Splitting](https://github.com/PeterCcT/trabalho-final-arquitetura/assets/72523562/15548566-b78e-4ef5-b32f-0a7217a43397)

## Escalabilidade

O discord foi criado, desde seu protótipo, em [Elixir](https://elixir-lang.org/): uma linguagem de programação funcional que é executada na máquina virtual “Erlang VM”. O uso dessa linguagem foi fundamental para permitir a alta escalabilidade do Discord, já que ela é uma linguagem moderna, amigável e desenvolvida para aplicações que visam alta escalabilidade, visto que possibilita a execução de múltiplos processos simultaneamente sem comprometer a perfomance da aplicação. Além disso, ela fornece para o desenvolvedor um ambiente bem distribuído que facilita muito a escalabilidade horizontal, ideal para sistemas de rápido crescimento.

Como o Discord é um aplicativo de comunicação entre usuários, a velocidade é um requisito fundamental, e essa também foi a maior barreira que os desenvolvedores enfrentaram para alcançar a alta escalabilidade desejada. Com o uso do Elixir, foi possível dividir os processos de envio de mensagens em diversas fases diferentes, ao invés de apenas uma, e com isso, foi possível reduzir pela metade o tempo de resposta que tinham. Com isso eles conseguiram escalar facilmente enquanto mantiveram a excelência do sistema.

Em relação aos serviços disponibilizados para os usuários, mais de 850 servidores para esses serviços são executados simultaneamente em 13 regiões, hospedados em mais de 30 centros de dados ao redor do mundo. Com essa arquitetura, o Discord pode atender mais de 2,6 milhões de usuários de voz simultâneos, com tráfego de saída de mais de 220 Gbps (bits por segundo) e 120 Mpps (pacotes por segundo).

## Tolerância a falhas

Com a disponibilização de 850 servidores distribuídos geograficamente mencionada anteriormente, o sistema fornece redundância para lidar com falhas e ataques DDoS a servidores (ateques que sobrecarregam a aplicação para deixar o serviço indisponível). Além disso, o a distribuição permite que os clientes tenham acesso aos serviços com uma latência menor.

A imagem abaixo ilustra a redundância do Discord Voice. Quando um servidor de voz do Discord falha, ele falha no ping periódico. O cliente percebe a falha do servidor devido à conexão WebSocket interrompida e solicita um ping de servidor de voz por meio da conexão WebSocket do gateway. O Discord Guilds confirma a falha e atribui um novo servidor de Discord Voice para a guilda. O Discord Guilds então envia todos os objetos de estado de voz para o novo servidor de voz. Todos os clientes são notificados sobre o novo servidor de voz e criam uma conexão WebSocket com o novo servidor.

![Tolerância a falhas](https://assets-global.website-files.com/5f9072399b2640f14d6a2bf4/612402ed42d2a34c22a0b203_1*cwbQeQWz8eofvEYpK_KwhA.gif)
## Referências

_BROWN, Abram. Discord Was Once The Alt-Right’s Favorite Chat App. Now It’s Gone Mainstream And Scored A New $3.5 Billion Valuation. Disponível em: https://www.forbes.com/sites/abrambrown/2020/06/30/discord-was-once-the-alt-rights-favorite-chat-app-now-its-gone-mainstream-and-scored-a-new-35-billion-valuation/. Acesso em maio de 2023_

_VASS, Jozsef. How Discord Handles Two and Half Million Concurrent Voice Users Using WebRTC. Disponível em: https://discord.com/blog/how-discord-handles-two-and-half-million-concurrent-voice-users-using-webrtc. Acesso em junho de 2023_

_HOWARTH, Jesse. How Discord Handles Push Request Bursts of Over a Million per Minute with Elixir's GenStage. Disponível em: https://discord.com/blog/how-discord-handles-push-request-bursts-of-over-a-million-per-minute-with-elixirs-genstage. Acesso em junho de 2023_

_GREER, Michael. How Discord Maintains Performance While Adding Features. Disponível em: https://discord.com/blog/how-discord-maintains-performance-while-adding-features. Acesso em junho de 2023_

_ARMSTRONG, Brian. How Discord Resizes 150 Million Images Every Day with Go and C. Disponível em: https://medium.com/discord-engineering/how-discord-resizes-150-million-images-every-day-with-go-and-c-c9e98731c65d. Acesso em junho de 2023_

_VISHNEVSKIY, Stanislav. How Discord Scaled Elixir to 5,000,000 Concurrent Users. Disponível em: https://discord.com/blog/how-discord-scaled-elixir-to-5-000-000-concurrent-users. Acesso em junho de 2023_

_CHEN, Donald. How Discord Made Android In-App Navigation Easier. Disponível em: https://discord.com/blog/how-discord-made-android-in-app-navigation-easier. Acesso em junho de 2023_

_VISHNEVSKIY, Stanislav. How Discord Stores Billions of Messages. Disponível em: https://discord.com/blog/how-discord-stores-billions-of-messages. Acesso em junho de 2023_

_INGRAM, Bo. How Discord Stores Trillions of Messages.Disponível em: https://discord.com/blog/how-discord-stores-trillions-of-messages. Acesso em junho de 2023_

_LAM, Sahn. How Discord Stores Trillions of Messages. Disponível em: https://www.youtube.com/watch?v=O3PwuzCvAjI&ab_channel=ByteByteGo. Acesso em junho de 2023_

_HEINZ, Jake. How Discord Indexes Billions of Messages. Disponível em: https://discord.com/blog/how-discord-indexes-billions-of-messages. Acesso em junho de 2023_
