---
layout: post
title:  "Introdução a Microserviços"
date:   2021-02-07 15:00:00 -0300
description: "O que é um Microserviço? Começamos nesse post a compreender a implementação de microserviços"
categories: 
  - api
  - microservice
---

![Img 01](https://raw.githubusercontent.com/daniloval/daniloval.github.io/main/_posts/2021-02-07-introducao-microservicos/IMG01.png)

Bem, projetar software é bastante complexo. Quando comecei a compreender as entranhas de minha primeira linguagem de programação (<s>Delphi 7, vê-se aqui a quanto tempo foi isso, rsrs</s>), foi bastante natural naquele momento ganhar confiança e me sentir cada vez mais orgulhoso sobre cada linha de código escrita, mas já naquele momento eu sabia que seria muito mais do que arrastar caixas de texto e implementar alguma regra de validação sobre aquele componente.

Conforme o tamanho e a complexidade dos projetos ao qual eu era inserido aumentavam, eu aprendia que nosso meio de trabalho é responsável por tomar decisões que vão muito além de simplesmente escrever linhas e mais linhas de código, estávamos na verdade transcrevendo histórias que outrora foram executadas por outros seres humanos de forma manual, modificando a maneira como tudo fora feito até ali, acelerando de forma imensurável mercados que nem imaginavam até aquele momento a capacidade de crescimento e revolução que a tecnologia nos trouxe. 

Lembro-me claramente que no começo da carreira eu me achava o desenvolvedor mais incrível do universo, e tinha certeza que era capaz de fazer qualquer coisa. Por uma dessas coincidências da vida, fui jogado quase que sem querer (podemos contar essa história por aqui em outro momento) em uma software house (Cadena) da minha cidade, foi um tremendo choque de realidade eu absolutamente não sabia de nada e obviamente não fiquei lá por mais de 3 meses isso me colocou na minha primeira crise do impostor (falaremos no futuro também) mas também me fez buscar mais e mais conhecimento, e entender naquele momento que estudar e testar coisas, provavelmente seria o que eu faria pela vida toda.

Durante os anos, novos projetos, times, empresas, e fui apresentado a um papel denominado "Arquiteto de Software", estes são responsáveis por conceber e delinear o panorama geral de um sistema, impactando diretamente (para melhor ou para pior) cada fase do ciclo de vida do projeto, desde o estágio de desenvolvimento até muito depois de seu lançamento final.

#### Arquitetura de Software

Ao longo dos anos, os arquitetos de software têm a incumbência de encontrar os melhores mecanismos para projetar sistemas que são escaláveis e de fácil manutenção. A arquitetura [N-Tier][1] é um exemplo famoso, apresentando uma segregação de responsabilidades que divide o código em pelo menos três camadas: dados, negócio e apresentação. Embora a princípio seja bom, a realidade nos mostra que é bastante difícil manter a camada de apresentação verdadeiramente impermeável. Sem um mecanismo para detectar quando esse princípio é violado, a lógica de negócios eventualmente vazará para a camada de apresentação, criando um acoplamento forte e um pesadelo quanto à sustentabilidade operacional.

A [arquitetura hexagonal][2] é um evolução disso, introduzindo o conceito de portas e adaptadores para impor um encapsulamento mais rígido e uma separação mais forte de interesses, encorajando os desenvolvedores a raciocinar sobre seus softwares como uma série de componentes modulares, em vez de uma pilha de camadas horizontais largas e pesadas. Mas mesmo assim, ainda estamos olhando para um grande aplicativo monolítico que precisa ser gerenciado e implantado como um todo, limitando a capacidade de comprometer efetivamente uma mudança em uma única parte do sistema com a confiança de que não afetará outras.

E finalmente, nos últimos anos, vemos um interesse crescente e cada vez mais difundido no paradigma de microserviços, que se caracteriza por promover componentes para serviços autônomos e implementáveis de forma independente. Os princípios básicos apresentados são vistos como uma forma robusta e confiável de sistemas de construção, e os principais participantes do setor estão usando-os em produtos de produção em larga escala.

Vamos tentar examinar por aqui todos os princípios, por meio de uma análise balanceada de seus benefícios e armadilhas encontradas, terminando com um visão técnica (com um viés bastante opinativo) sobre por que microserviços e Node.js é uma combinação um tanto quanto atraente.

#### O Monolito

Em uma abordagem tradicional, normalmente possuímos um único software formado por uma base de código dividida internamente nas seguintes camadas:
<ul>
<li>Camada de apresentação, usada para exibir o conteúdo aos usuários, também potencialmente recebendo entrada (no caso de um aplicativo da web, isso seria um conjunto de páginas HTML);</li>
<li>Camada lógica ou de negócios, responsável por todo o processamento intermediário, executando a lógica que é específica para o domínio de negócios;</li>
<li>Camada de acesso a dados, lê e grava dados em banco de dados.</li>
</ul>

Os vários componentes que formam cada uma dessas camadas são agrupados e implantados em um servidor de produção como uma entidade única e unitária.

Vamos imaginar o software de blog construído com essa arquitetura, usando um banco de dados relacional como armazenamento de dados. Se tivéssemos de renderizar uma página com um post específica junto com todos os comentários gerados pelos usuários vinculados a ele, o software consultaria as tabelas de posts e comentários, usando um SQL JOIN ou semelhante, processa e formata os dados conforme necessário e exibir de volta ao cliente. Toda essa comunicação e processamento acontece por meio de chamadas ou requisição única.

![Img 02](https://raw.githubusercontent.com/daniloval/daniloval.github.io/main/_posts/2021-02-07-introducao-microservicos/IMG02.png)

Então, o que exatamente são microserviços e como eles se diferenciam do paradigma monolítico? Sempre que alguém me faz  essa pergunta ou quando estou fazendo uma palestra sobre o assunto, refiro-me a esta citação de Martin Fowler:

"O estilo de arquitetura de microserviços é uma abordagem para desenvolver um único software com um pacote de pequenos serviços, cada um executando em seu próprio processo e se comunicando com mecanismos leves, geralmente uma API de recurso HTTP. Esses serviços são desenvolvidos em torno de recursos de negócios e podem ser implantados de forma independente por meio de máquinas de implantação totalmente automatizadas."

Com microserviços, os componentes não vivem juntos dentro dos limites de uma única unidade. Em vez disso, essa arquitetura promove o desenvolvimento e a implantação de aplicativos como um conjunto de serviços independentes e autocontidos. Se modificarmos o exemplo acima usando esse estilo de arquitetura, as etapas que seguiríamos para obter o mesmo conteúdo poderiam ser bem diferentes.

Em vez de possuirmos o software consultando um único banco de dados para posts e comentários, poderíamos ter um serviço exclusivo para posts, responsável por armazenar, gerenciar e entregar histórias. O serviço seria executado em seu próprio processo e todas as comunicações de e para ele são estabelecidas por meio de chamadas de rede, processadas por seu próprio servidor web e usando uma API que expõe ao mundo exterior. Se o próprio serviço precisar fazer uso de outros serviços, isso também acontece por meio de chamadas de rede (com exceção de uma conexão a um banco de dados, que pode ou não ser uma chamada direta em processo). Os mesmos princípios também podem ser aplicados ao um serviço de comentários, conforme imagem abaixo.

![Img 03](https://raw.githubusercontent.com/daniloval/daniloval.github.io/main/_posts/2021-02-07-introducao-microservicos/IMG03.png)

Sabemos que microserviços se referem à construção de um sistema por meio da composição de vários serviços independentes, mas o que mais define essa arquitetura? Abordaremos os princípios chave dos microserviços e os benefícios associados que eles podem trazer para um software no próximo post.

Gostou desse post? Considere assinar a Newsletter logo ai abaixo.

<blockquote>Vem comigo, que no caminho eu te explico...</blockquote>


 [1]: https://docs.microsoft.com/pt-br/previous-versions/visualstudio/visual-studio-2015/data-tools/n-tier-data-applications-overview?view=vs-2015&redirectedfrom=MSDN
 [2]: https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)