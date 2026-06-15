# API vs raw.githubusercontent.com: Por Que o Canal de Leitura Importa

**Capítulo:** 04
**Data:** 2026-06-16
**Categoria:** Boas práticas operacionais — acesso a dados
**Origem:** Curadoria T1 (W04, 2026-06-08); consolidação 2026-06-16

---

## O problema que não víamos

No início da construção do ecossistema, a regra de leitura era simples e
intuitiva: para ler um arquivo do GitHub, bastava buscar sua URL pública em
`raw.githubusercontent.com`. Era rápido, não exigia autenticação, e funcionava
na maioria das vezes. *Na maioria das vezes* — essa é a parte que cobrou seu
preço.

O sintoma apareceu de forma traiçoeira: depois de escrever um arquivo via API
(um `PUT` bem-sucedido, HTTP 200, confirmação no corpo da resposta), uma leitura
imediata pelo canal raw às vezes devolvia a versão *anterior* do arquivo — como
se a escrita não tivesse acontecido. O agente, lendo conteúdo obsoleto, tomava
decisões sobre um estado do ecossistema que já não existia. Em um sistema cujo
princípio central é "nenhum arquivo fica desatualizado por esquecimento", ler
o passado achando que é o presente é a falha mais perigosa possível.

## A causa raiz

A causa não era um bug nosso. Era a natureza dos dois canais.

`raw.githubusercontent.com` é servido através de uma **CDN** (Content Delivery
Network) com cache. A CDN existe para entregar conteúdo público com rapidez e
escala — e faz isso guardando cópias do arquivo em pontos de presença
distribuídos. Quando um arquivo é alterado, há uma janela de propagação durante
a qual o cache ainda serve a versão antiga. Essa janela é **indeterminada**:
pode ser instantânea ou durar minutos. Para leitura de conteúdo público estável,
isso é irrelevante. Para leitura logo após uma escrita, é veneno.

A **GitHub Contents API** (`api.github.com`) não passa por esse cache de CDN.
Ela responde a partir do estado consistente do repositório, devolvendo o
conteúdo em base64 junto ao SHA atual do arquivo. O custo é exigir autenticação
e consumir cota de requisições — mas a recompensa é a consistência: o que a API
devolve é o que está no repositório *agora*.

Havia ainda um segundo fator, que demorou a entrar em foco: o canal raw **não lê
repositórios privados**. À medida que o ecossistema cresceu e ganhou repositórios
privados (o handoff no `hub-memoria`, agendas, projetos), o canal raw deixou de
ser sequer capaz de cumprir o papel — independentemente do problema de cache.

## A regra que adotamos

A lição cristalizou em uma instrução operacional simples:

> **Em sessão de trabalho, leia sempre via API GitHub autenticada — nunca via
> `raw.githubusercontent.com`.**

O raw foi aposentado como canal de sessão. Permanece apenas como recurso público
de leitura sem token para repositórios públicos, fora do rito de sessão — por
exemplo, demonstrar o ecossistema a um terceiro que não tem token. Mas nada que
envolva operar sobre o estado do ecossistema, e nada que toque repositório
privado, usa raw.

Essa regra mais tarde se desdobrou na **doutrina de dois tokens**: token de
leitura ampla na abertura (alcança privados, eleva o teto de requisições, elimina
o cache CDN) e token de edição apenas na conversão para escrita.

## O que o mercado chama isso

O fenômeno que enfrentamos tem nome consagrado na literatura de sistemas
distribuídos: **cache coherence** e o problema de **read-after-write
consistency**. Um sistema oferece consistência *read-after-write* quando garante
que uma leitura realizada após uma escrita bem-sucedida sempre reflete aquela
escrita. CDNs, por design, sacrificam essa garantia em favor de latência e
escala — operam sob **consistência eventual** (*eventual consistency*): a réplica
converge para o valor correto, mas não imediatamente.

A escolha entre os dois canais é, portanto, uma escolha clássica entre dois
pontos do espectro de consistência: o raw otimiza disponibilidade e latência ao
custo de consistência; a API prioriza consistência ao custo de autenticação e
cota. Para um sistema de governança documental, onde o estado precisa ser
verdadeiro no instante da leitura, a consistência não é negociável.

## A lição permanente

Quando um sistema oferece mais de um canal para acessar o mesmo dado, esses
canais quase nunca são equivalentes — eles fazem trade-offs diferentes. O canal
mais rápido e conveniente costuma ser o que relaxa alguma garantia. A pergunta
que aprendemos a fazer não é *"qual canal é mais fácil?"*, mas *"qual garantia
este canal me dá, e essa garantia é a que minha operação exige?"*.

Para leitura de estado operacional sobre o qual vamos agir, a garantia exigida é
consistência read-after-write. Só a API a oferece. A conveniência do raw era uma
economia falsa: pagávamos depois, em decisões tomadas sobre um passado disfarçado
de presente.

---

*Capítulo registrado a partir da curadoria T1, identificada na primeira sessão
de roadmapping (W04, 2026-06-08) e consolidada em 2026-06-16. Memória intelectual
do ecossistema DTD/SETIS.*
