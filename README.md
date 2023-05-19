## Performance

O Discord mantém a performance de sua aplicação mesmo após a evolução de seu software. Mas como isso é feito? A equipe de desenvolvimento utiliza uma abordagem iterativa e focada na melhoria contínua.

Uma das estratégias adotadas pela equipe de desenvolvimento do Discord para manter a performance da aplicação é a otimização do código e dos algoritmos utilizados. Eles procuram constantemente maneiras de tornar o software mais eficiente e rápido, reduzindo o tempo de carregamento e resposta em diferentes áreas da plataforma.

Além disso, o Discord faz uso de técnicas de escalabilidade, o que significa que a aplicação é capaz de lidar com um aumento significativo no número de usuários sem comprometer sua performance. Isso é especialmente importante para uma plataforma de chat e comunicação em tempo real como o Discord, que precisa suportar uma grande quantidade de mensagens e interações simultâneas.

A equipe de desenvolvimento também realiza monitoramento constante do desempenho da aplicação, utilizando métricas e ferramentas de análise para identificar possíveis gargalos e problemas de desempenho. Dessa forma, eles podem agir rapidamente para corrigir essas questões e melhorar a experiência do usuário.

No geral, o Discord adota uma abordagem abrangente para garantir a performance de sua aplicação, combinando otimizações técnicas, arquitetura escalável, monitoramento constante e testes rigorosos. Isso permite que os usuários desfrutem de uma experiência suave e responsiva ao utilizar a plataforma.

## Escalabilidade

O discord foi criado, desde seu protótipo, em [Elixir](https://elixir-lang.org/), uma linguagem de programação funcional que é executada na máquina virtual “Erlang VM”. O uso dessa linguagem foi fundamental para permitir a alta escalabilidade do Discord, já que ela é uma linguagem moderna, amigável e desenvolvida para aplicações que visam alta escalabilidade, por possibilitar a execução de múltiplos processos simultaneamente sem comprometer a perfomance da aplicação. Além disso, ela fornece para o desenvolvedor um ambiente bem destribuído que facilita muito a escalabilidade horizontal, ideal para sistemas de rápido crescimento. 

Como o Discord é um aplicativo de comunicação entre usuários, a velocidade é um requisito fundamental, e essa também foi a maior barreira que os desenvolvedores enfrentaram para alcançar a alta escalabilidade desejada. Com o uso do Elixir, foi possível dividir os processos de envio de mensagens em diversas fases diferentes, ao invés de apenas uma, e com isso, foi possível reduzir pela metade o tempo de resposta que tinham. Com isso eles conseguiram escalar facilmente enquanto mantiveram a excelência do sistema.

Em relação aos serviços disponibilizados para os usuários, mais de 850 servidores para esses serviços são executados simultaneamente em 13 regiões, hospedados em mais de 30 centros de dados ao redor do mundo. Com essa arquitetura, o Discord pode atender mais de 2,6 milhões de usuários de voz simultâneos, com tráfego de saída de mais de 220 Gbps (bits por segundo) e 120 Mpps (pacotes por segundo).

## Tolerância a falhas

Com a disponibilização de 850 servidores distribuídos geograficamente mencionada anteriormente, o sistema fornece redundância para lidar com falhas e ataques DDoS a servidores (ateques que sobrecarregam a aplicação para deixar o serviço indisponível).

## Referências

Discord Blog. Disponível em: https://discord.com/blog/. Acesso em maio de 2023

