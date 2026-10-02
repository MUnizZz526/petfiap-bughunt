# Checkpoint 5 — Bug Hunt PetFiap


## Identificação

**Grupo:** Debugadores FIAP

| Integrante | RM | Turma |
|---|---|---|
| Cauã Muniz | RM566527 | 2CCPH |
| Matheus Morelli | RM562765 | 2CCPH |
| Victor Mortari | RM564804 | 2CCPH |
| Henrique Mortari | RM564699 | 2CCPH |
|Rafael Ferreira | RM563285 | 2CCPH |


| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | 26 testes, 0 falhas |

---

## Parte 1 — Bugs encontrados

|---|---|---|---|---|
| bug01 | `GeradorProtocoloTest.deveManterUmaUnicaInstancia` e `deveGerarProtocolosSequenciais` falhavam: criava novas instâncias e o contador reiniciava em 1 | `GeradorProtocolo.java:18-20` — método `getInstancia()` retornava `new GeradorProtocolo()` sem salvar na variável estática `instancia` | Atribuir `instancia = new GeradorProtocolo()` antes de retornar a variável estática | Padrão Singleton e membros estáticos (Aula 14) |
| bug02 | `AtendimentoBuilderTest.deveMontarAtendimentoCompleto` falhava com `expected: <Rex> but was: <null>` | `AtendimentoBuilder.java:24` — no método `comPet`, a variável `petNome = petNome` sofria sombreamento de escopo sem o prefixo `this.` | Corrigir a atribuição para `this.petNome = petNome;` | Escopo de variáveis, sombreamento (shadowing) e operador `this` (Aula 2/3) |
| bug03 | `AtendimentoBuilderTest.deveRecusarMontagemSemNomeDoPet` falhava porque não lançava `IllegalArgumentException` quando o nome do pet era nulo | `AtendimentoBuilder.java:41-43` — método `construir` não validava os campos obrigatórios antes de acionar a factory | Adicionar validação no `construir`: lançar `IllegalArgumentException` caso `petNome == null \|\| petNome.isBlank()` | Padrão Builder e princípio de invariantes ("o objeto só nasce válido") (Aulas 11 e 14) |
| bug04 | `AtendimentoBuilderTest.deveRecusarMontagemSemPorte` falhava porque não lançava `IllegalArgumentException` quando o porte do pet era nulo | `AtendimentoBuilder.java:41-43` — método `construir` não validava a presença obrigatória do porte do animal | Adicionar validação no `construir`: lançar `IllegalArgumentException` caso `petPorte == null \|\| petPorte.isBlank()` | Padrão Builder e consistência de dados do domínio (Aulas 11 e 14) |
| bug05 | `AtendimentoFactoryTest.deveCriarTosaQuandoTipoForTosa` falhava com `expected: <...Tosa> but was: <...Banho>` | `AtendimentoFactory.java:17` — no `switch`, o `case "TOSA"` instanciava `new Banho(...)` em vez de `new Tosa(...)` | Alterar o retorno da opção `"TOSA"` para `new Tosa(protocolo, petNome, petPorte, tutorNome, dataHora)` | Padrão Factory e Polimorfismo (Aula 14) |
| bug06 | `AtendimentoFactoryTest.devePreencherOsDadosDoPetNaConsulta` falhava com `expected: <Mimi> but was: <null>` | `ConsultaVeterinaria.java:17` — construtor chamava `super();` sem repassar os parâmetros recebidos para a superclasse `Atendimento` | Corrigir chamada do construtor da superclasse para `super(protocolo, petNome, petPorte, tutorNome, dataHora)` | Herança, encapsulamento e construtores de subclasses (`super`) (Aula 6/7) |
| bug07 | `AgendaServiceTest.deveRecusarAgendamentoComHorarioJaOcupado` falhava com `NullPointerException` (não detectava conflito de horário agendado) | `AgendaService.java:23` — comparava instâncias de `LocalDateTime` e `String` utilizando o operador referencial `==` em vez de `.equals()` | Substituir a comparação referencial por `.equals()` para `petNome` e `dataHora` | Comparação de igualdade de objetos em Java (`==` vs `.equals()`) (Aula 7) |
| bug08 | `AgendaServiceTest.deveLancarExcecaoQuandoAtendimentoNaoExiste` falhava pois retornava `null` sem lançar a exceção esperada | `AgendaService.java:37-42` — bloco `try-catch` genérico capturava `Exception` e retornava `null`, engolindo o `orElseThrow` | Remover o bloco `try-catch`, propagando a `AtendimentoNaoEncontradoException` diretamente para o chamador | Tratamento de exceções e anti-padrão de engolir exceções (Aula 11) |
| bug09 | Banho para pet de porte pequeno custava R$ 100,00 e porte grande custava R$ 60,00, contrariando a tabela oficial do contrato | `Banho.java:27-32` — método `calcularPreco()` continha valores invertidos para os portes PEQUENO e GRANDE | Corrigir a lógica de cálculo: PEQUENO = 60.0, MEDIO = 80.0 e GRANDE = 100.0 | Polimorfismo e regras de negócio encapsuladas no Model (Aula 6/7) |
| bug10 | Atendimento de tosa retornava duração de 30 minutos (default de `Atendimento`) em vez dos 60 minutos contratuais | `Tosa.java:40` — método declarado como `getDuracaoMinutos(String porte)`, criando sobrecarga (overload) em vez de sobrescrita (override) | Remover o parâmetro `String porte` e anotar com `@Override public int getDuracaoMinutos()` retornando 60 | Sobrescrita (@Override) vs Sobrecarga (Overload) de métodos (Aula 7) |
| bug11 | `AgendaService.agendar` aceitava agendamentos com data e hora no passado, persistindo dados retroativos indevidos | `AgendaService.java:20-29` — ausência de verificação temporal antes de consultar o banco de dados | Adicionar validação no início de `agendar`: lançar `IllegalArgumentException` caso `dataHora` seja anterior a `LocalDateTime.now()` | Validação de regras de negócio na camada de serviço (Aulas 11 e 13) |
| bug12 | Método `cancelar()` permitia cancelar atendimentos já concluídos ou cancelados, violando a máquina de estados | `Atendimento.java:63-65` — método `cancelar()` alterava o status para CANCELADO incondicionalmente sem checagem de estado | Adicionar validação: se `!"AGENDADO".equals(status)`, lançar `StatusInvalidoException` | Encapsulamento, invariantes de estado e exceções de domínio (Aulas 5 e 11) |

---

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoFactory.java` (assinatura do método `criar`) | Nomes significativos e legibilidade (Clean Code Cap. 2). Uso de parâmetros abreviados de uma única letra (`p, t, n, po, tu, d`) | Renomeados os parâmetros para nomes autoexplicativos: `protocolo, tipo, petNome, petPorte, tutorNome, dataHora` |
| clean02 | `GeradorProtocolo.java` (construtor privado) | Poluição de saída padrão e uso indevido de `System.out.println` em código de produção | Removida a linha `System.out.println("GeradorProtocolo criado!");` |
| clean03 | `AgendaService.java` (método `agendar`) | Efeitos colaterais de I/O desnecessários e uso de `System.out.println` na camada de serviço | Removida a linha de impressão de recibo no console (`System.out.println(...)`) |
| clean04 | `AgendaService.java` (atributo `repository`) | Injeção de dependência por atributo (`@Autowired` no field) fere o princípio de imutabilidade e dificulta testes unitários manuais | Refatorado para injeção via construtor com o atributo definido como `private final` |
| clean05 | `AtendimentoController.java` (atributo `service`) | Injeção de dependência por atributo (`@Autowired` no field) na camada de controle web | Refatorado para injeção via construtor com o atributo definido como `private final` |
| clean06 | `AtendimentoController.java` (linhas 110-116) | Código morto (Dead Code) e comentários especulativos sobre regras de negócio futuras (Clean Code Cap. 17 / YAGNI) | Removido o método privado não utilizado `calcularDescontoFidelidade` e os comentários de especulação |

---

## Parte 3 — Testes novos (regras que estavam sem cobertura)

> Uma linha por teste novo (`test: teste01 ...` a `test: teste06 ...`), cobrindo as 6 regras de negócio contratuais que não possuíam cobertura na suíte inicial.

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `BanhoTest.deveCustar60ReaisParaPortePequeno` | Banho para pet de porte PEQUENO deve custar R$ 60,00 segundo a tabela de preços | Vermelho (revelou o bug09: retornava R$ 100,00) |
| teste02 | `TosaTest.deveDurar60Minutos` | Atendimento do tipo TOSA deve ter duração de 60 minutos | Vermelho (revelou o bug10: retornava 30 minutos por sobrecarga) |
| teste03 | `AgendaServiceTest.deveRecusarAgendamentoComDataNoPassado` | Agendamento com data/hora no passado deve lançar `IllegalArgumentException` sem consultar o banco | Vermelho (revelou o bug11: agendamento retroativo era aceito) |
| teste04 | `AgendaServiceTest.deveRecusarCancelamentoDeAtendimentoJaConcluido` | Cancelamento de atendimento já com status CONCLUIDO deve recusar lançando `StatusInvalidoException` | Vermelho (revelou o bug12: cancelava sem validar status anterior) |
| teste05 | `ConsultaVeterinariaTest.deveCustar150ReaisIndependenteDoPorte` | Consulta veterinária possui preço fixo de R$ 150,00 independente do porte do animal | Verde de cara (regra já estava implementada corretamente no Model) |
| teste06 | `AgendaServiceTest.deveCancelarAtendimentoAgendado` | Cancelamento de atendimento agendado (caminho feliz) deve alterar o status para CANCELADO e salvar | Verde de cara (caminho feliz de cancelamento com status AGENDADO já funcionava) |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)
Ao executar a suíte original, os 9 testes vermelhos forneceram relatórios de falha assertivos que direcionaram a investigação para a raiz exata de cada problema. Por exemplo, a falha `expected: <Rex> but was: <null>` em `AtendimentoBuilderTest.deveMontarAtendimentoCompleto` evidenciou de imediato que o atributo `petNome` não estava sendo preenchido no builder, levando diretamente ao bug de autoatribuição `petNome = petNome` na linha 24. De forma análoga, o erro `expected: <...Tosa> but was: <...Banho>` no `AtendimentoFactoryTest` revelou que o switch da factory instanciou a classe incorreta. 

A suíte automatizada é incomparavelmente superior aos testes manuais com `curl` porque roda em menos de 2 segundos, não depende de infraestrutura externa (como banco Oracle ativo ou portas de rede livres), isola cada componente por testes unitários e oferece feedback imediato sobre regressões: qualquer alteração no código que quebre um contrato pré-estabelecido é detectada instantaneamente sem o trabalho braçal de montar requisições HTTP manuais.

### 2. Mock e injeção de dependência (Aulas 13 a 15)
Em produção, a injeção de dependências é governada pelo contêiner IoC do Spring Framework através da anotação `@Autowired` (ou via construtor), que resolve e injeta a instância real gerenciada de `AtendimentoRepository`, conectando-se fisicamente ao banco de dados Oracle da FIAP através do Spring Data JPA / Hibernate. 

Já no ambiente de testes unitários (`AgendaServiceTest`), o contêiner do Spring sequer é inicializado: quem realiza a injeção é o Mockito através da extensão `@ExtendWith(MockitoExtension.class)`. A anotação `@Mock` cria um proxy dinâmico (objeto simulado em memória) de `AtendimentoRepository`, e a anotação `@InjectMocks` o injeta diretamente no construtor do `AgendaService`. Dessa forma, o teste unitário executa de maneira estritamente determinística e ultrarrápida sem necessidade de conexão de rede ou banco de dados relacional, pois os comportamentos do repositório (como `findByPetNome` e `save`) são previamente instruídos com chamadas `when(...).thenReturn(...)` simuladas em memória.

### 3. `==` vs `.equals()` (Aula 7)
Em Java, o operador `==` compara a identidade física de referências de memória (se dois ponteiros apontam para o mesmíssimo endereço no Heap), ao passo que o método `.equals()` compara a igualdade lógica dos valores representados pelos objetos. No método `AgendaService.agendar`, a verificação de conflito utilizava `a.getDataHora() == novo.getDataHora()`. Essa comparação falhava porque instâncias distintas de `LocalDateTime` criadas em momentos diferentes — mesmo com mesmo ano, mês, dia, hora e minuto — residem em regiões de memória diferentes, fazendo o `==` avaliar como `false`. 

O código parecia "funcionar por sorte" para o nome do pet quando se utilizavam literais idênticos como `"Rex"` devido ao mecanismo de String Constant Pool da JVM, que otimiza literais repetidos no mesmo endereço. Porém, se a String viesse instanciada dinamicamente via request HTTP ou `new String()`, a comparação também falharia. A correção consistiu em aplicar `.equals()` tanto para `petNome` quanto para `dataHora`, assegurando que o conflito de agenda seja detectado com base no valor lógico dos dados.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Sobrescrita (Override) ocorre quando uma subclasse implementa um método já existente na superclasse preservando estritamente a mesma assinatura (mesmo nome, tipos de parâmetros e retorno compatível), habilitando o polimorfismo em tempo de execução. Sobrecarga (Overload), por sua vez, acontece quando métodos compartilham o mesmo nome, porém possuem assinaturas distintas (quantidade ou tipos de parâmetros diferentes), configurando métodos completamente independentes para a linguagem. 

Na classe `Tosa`, o estagiário escreveu `public int getDuracaoMinutos(String porte) { return 60; }`. Por possuir o parâmetro extra `String porte`, esse método não sobrescreveu o `getDuracaoMinutos()` herdado de `Atendimento`, mas sim criou uma sobrecarga. O código compilava sem alertas, mas, em tempo de execução, chamadas polimórficas `tosa.getDuracaoMinutos()` acionavam o método da superclasse, retornando 30 minutos em vez de 60. O uso da anotação `@Override` teria impedido o bug em tempo de compilação, pois o compilador Java acusaria erro de compilação imediato indicando que nenhum método com aquela assinatura existia na classe ancestral.

### 5. Singleton manual vs bean do Spring (Aula 14)
O padrão Singleton garante a existência de uma única instância de uma classe em toda a JVM durante a execução da aplicação, fornecendo um ponto global de acesso seguro a esse objeto. Em nosso sistema, o `GeradorProtocolo` tem a responsabilidade de manter e incrementar a sequência global de protocolos dos atendimentos através do método `proximo()`. O bug ocorreu porque a implementação manual dentro de `getInstancia()` continha `if (instancia == null) { return new GeradorProtocolo(); }`, instanciando um novo objeto a cada invocação sem atribuí-lo à variável estática `instancia`. Com isso, cada atendimento recebia um gerador zerado, gerando sempre o protocolo 1. 

O `AgendaService`, por ser anotado com `@Service`, não corre esse risco porque seu ciclo de vida é administrado pelo contêiner de Inversão de Controle (IoC) do Spring Boot. Por padrão, todos os beans gerenciados pelo Spring operam no escopo Singleton: o contêiner instancia o serviço uma única vez durante a inicialização do contexto da aplicação e o injeta de forma centralizada e thread-safe em todos os pontos necessários, eliminando o código boilerplate e os erros clássicos de verificação estática manual.

### 6. Cobertura de testes: onde parar? (Aula 15)
Vale absolutamente a pena manter os testes que ficaram verdes de cara (como `ConsultaVeterinariaTest.deveCustar150ReaisIndependenteDoPorte` e `AgendaServiceTest.deveCancelarAtendimentoAgendado`). A finalidade dos testes de software vai além de encontrar falhas no momento de sua criação; sua função primordial a longo prazo é servir como uma suíte de regressão viva e documentação executável que blinda o sistema contra alterações indevidas em refatorações futuras. 

Em um cenário corporativo real com restrição de prazo, a busca obstinada por 100% de cobertura métrica é contraproducente, pois gera desperdício de tempo testando getters, setters e detalhes triviais de frameworks. A prioridade estratégica de testes deve se concentrar nos caminhos felizes dos fluxos principais que entregam valor direto ao negócio (como o fluxo completo de agendamento e conclusão), seguidos imediatamente pelos fluxos de exceção críticos e regras de integridade (como validação de datas no passado, regras de transição de status e prevenção de agendamentos conflitantes). Testar os cenários de maior impacto econômico e de integridade de dados garante máxima confiabilidade do produto com o melhor retorno sobre o investimento de desenvolvimento.

---

