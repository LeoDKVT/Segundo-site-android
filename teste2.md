### Simplificando a Implicidade

Antes até de nós chegarmos ao JavaScript, deixe-me sugerir algum pseudo-código de uma linguagem teórica fortemente tipada para ilustrar:

```js
SomeType x = SomeType( AnotherType( y ) )
```

Nesse exemplo, eu tenho tenho um tipo de valor arbitrário em `y` que eu quero converter para o tipo `SomeType`. O problema é, essa linguagem não pode ir diretamente de qualquer coisa que `y` é pra `SomeType`. Ele precisa de um passo intermediário, onde ele primeiro converte para `AnotherType`, e então de `AnotherType` para `SomeType`.

Agora, e se a linguagem (ou definição que você mesmo pode criar com a linguagem) deixasse você *dizer*:

```js
SomeType x = SomeType( y )
```

Você não concorda que nós simplificamos o tipo de conversão aqui para reduzir o "ruído" desnecessário do passo de conversão intermediária? Quero dizer, isso é *realmente* tão importante, aqui mesmo nesse ponto do código, para ver e lidar com o fato que `y` vai primeiro para `AnotherType` antes e então vai para `SomeType`?

Alguns argumentariam, pelo menos em algumas circunstâncias, sim. Mas eu acho que um argumento equivalente pode ser feito de várias outras cinscunstâncias que aqui, a simplificação **na verdade ajuda na legibilidade do código** absorvendo ou escondendo tais detalhes, tanto na própria linguagem ou nas suas próprias abstrações.

Sem dúvidas, nos bastidores, em algum lugar, a conversão intermediária continua acontecendo. Mas se esse detalhe é oculto da view, nós apenas podemos raciocinar sobre pegar `y` para o tipo `SomeType` como uma operação genérica e enconder os detalhes bangunçados.

Embora não seja uma analogia perfeita, o que eu vou argumentar em todo o resto desse capítulo é que coerção *implícita* JS pode ser considerada como fornecedora de uma ajuda similar para seu código.

Mas, **e isso é muito importante**, essa não é uma declaração absoluta e ilimitada. Há definitivamente uma abundância de *males* que espreitam a coerção *implícita*, que prejudicará seu código muito mais do que qualquer potencial melhoria de legibilidade. Claramente, nós teremos que aprender como evitar certas construções para que não envenenemos nosso código com todas as formas de bugs.

Muitos desenvolvedores acreditam que se um mecanismo pode fazer algo últil **A** mas também pode ser abusado ou mal usado para fazer algo terrível **Z**, então nós devemos descartar completamente esse mecanismo, apenas por segurança.

Meu conselho para você é: não se conforme com isso. Não "mate uma mosca com uma bala de canhão". Não assuma coerção *implícita* é de toda ruim porque tudo que você acha que já viu são "partes ruins". Eu penso que há "partes boas" aqui, e eu quero ajudar e inspirar você para acha-las e absorve-las.

### Implicitamente: Strings <--> Numbers

Mais cedo nesse capítulo, nós exploramos a coerção *implícita* entre valores `string` e `number`. Agora, vamos explorar a mesma tarefa mas com abordagem de coerção *implícita*. Mas antes, nós temos que examinar algumas nuances de operações que vão forçar a coerção *implícita*.

O operador `+` é encarregado de servir propósitos tanto de adição de `number` como concatenação de `string`. Então como o JS sabe qual tipo de operação você quer usar? Considere:

```js
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42
```

Qual a diferença que causa `"420"` vs `42`? É um equívoco comum que a diferença é se um ou ambos os operadores são uma `string`, pois isso significa que `+` assumirá a concatenação `string`. Enquando isso é parcialmente verdade, é mais complicado que isso.

Considere:

```js
var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"
```

Nenhum desses operandos é uma `string`, mas claramente ambos sofrem coerção para `string`s e então a concatenação `string` pula dentro. Então o que realmente stá acontecendo?

(**Atenção**: terrível e profunda linguagem de especificação abaixo, então pule os próximos dois parágrafos se isso intimida você!)

-----

De acordo com  a seção 11.6.1 da especificação ES5, o algoritmo `+` (quando um valor `object` é um operando) vai concatenar se um dos operandos já for uma `string`, ou se os passos seguintes produzirem uma representação `string`. Então quando o `+` recebe um `object` (incluindo `array`) para ambos operandos, ele primeiro chama a operação abstrata `ToPrimitive` (seção 9.1) no valor, o que então chama o algoritmo `[[DefaultValue]]` (section 8.12.8) com um contexto `number`.

Se você está prestando bastante atenção, você irá notar que essa operação é agora indêntica a como a operação abstrata `ToNumber` maneja `object` (veja a seção anterior "`ToNumber`"). A operação `valueOf()` no `array` vai falhar em produzir um primitivo simples, então ela cai na representação `toString()`. Os dois `array`s irão então se tornar `"1,2"` and `"3,4"`, respectivamente. Agora, `+` concatena as duas `string` como você espera: `"1,23,4"`.

-----

Vamos olhar além desses detalhes confusos e voltar para uma explicação simplificada: se o operando para `+` é uma `string` (ou torna-se uma com os passos acima!), a operação será concatenação de `string`. Do contrário, ela sempre será adição numérica.

**Observação** uma pegadinha de coerção comumente citada é `[] + {}` vs. `{} + []`, como essas duas expressões resultam, repectivamente, `"[object Object]"` e `0`. Há ainda mais, e cobrimos esses detalhes em "Blocos" no capítulo 5.

O que isso significa para coerção *implícita*?

Voce pode fazer a coerção de um `number` para uma `string` simplismente "adicionando" o `number` e a `string` vazia `""`:

```js
var a = 42;
var b = a + "";

b; // "42"
```

**Dica:** Adições numéricas com o operador `+` é comutativa, o que significa que `2 + 3` é o mesmo que `3 + 2`. Concatenação de String com `+` obviamente não é comutativa geralmente, **mas** com o caso específico do `""`, ela é efetivamente comutativa, assim como `a + ""` e `"" + a` irão produzir o mesmo resultado.

É extremamente comum/idiomático fazer a coerção (*implicitamente*) de `number` para `string` com uma operação `+""`. De fato, é interessante que, mesmo alguns dos maiores críticos da coerção *implícita* ainda usam essa abordagem em seu próprio código, em vez de uma das suas alternativas *explícitas*.

**Eu acho que esse é um grande exemplo** de formas úteis na coerção *implícita*, apesar do quão frequentemente o mecanismo recebe críticas.

Comparando essa coerção *implícita* de `a + ""` com nosso exemplo anterior de coerção *explícita* `String(a)`, há uma peculiaridade adicional para ter cuidado. Por causa de como a operação abstrata `ToPrimitive` funciona, `a + ""` invoca `valueOf()` no valor de `a`, no qual o valor de retorno é então finalmente convertido para uma `string` via operação abstrata interna `ToString`. Mas `String(a)` apenas invoca `toString()` diretamente.

Ambas abordagens vão resultar em uma `string` no final, mas se você está usando um `object` em vez de um valor primitivo `number` normal, você pode, não necessariamente, ter o *mesmo* valor de `string`!

Considere:

```js
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"
```

Geralmente, esse tipo de pegadinha não vai te pegar a menos que você realmente esteja tentando criar estruturas de dados e operações confusas, mas você deve ter cuidado se você está definindo métodos próprios `valueOf()` e `toString()` para algum `object`, pois a forma de fazer a coerção pode afetar o resultado.

E a outra direção? Como podemos fazer a *coerção implícita* de `string` para `number`?

```js
var a = "3.14";
var b = a - 0;

b; // 3.14
```

O operador `-` é definido apenas para subtrações numéricas, então `a - 0` força o valor `a` a sofrer coerção para `number`. Embora muito menos comum, `a * 1` or `a / 1` realizarão o mesmo resultado, já que esses operadores também são definidos apenas para operações numéricas.

E os valores `object` com o operador `-`? Mesma história que para o `+` acima:

```js
var a = [3];
var b = [1];

a - b; // 2
```

Ambos valores `array` precisam se tornar `number`s, mas eles terminam primeiro sofrendo a coerção para `string` (usando a serialização esperada `toString()`), e então sofrem a coerção para `number`s, para a substração `-` seja aplicada.

Então, a coerção *implícita* de valores `string` e `number` são tão maléficas sobre a qual você sempre ouviu histórias de terror? Pessoalmente, eu não acho.

Compare `b = String(a)` (*explícita*) com `b = a + ""` (*implícita*). Eu acho que casos podem ser feitos para que ambas abordagens sejam úteis para seu código. Certamente `b = a + ""` é um pouco mais comum em programas JS, provendo sua própria utilidade independentemente de *sentimentos* sobre os méritos e perigos da coerção *implícita* em geral.

### Implicitamente: Booleans --> Numbers

Eu acho que um caso onde coerção *implícita* pode realmente brilhar é em simplificar certos tipos de lógicas `boolean` complicadas em simples adições numéricas. Claro, essa não é uma técnica com propósito geral, mas uma solução específica para casos específicos.

Considere:

```js
function onlyOne(a,b,c) {
	return !!((a && !b && !c) ||
		(!a && b && !c) || (!a && !b && c));
}

var a = true;
var b = false;

onlyOne( a, b, b );	// true
onlyOne( b, a, b );	// true

onlyOne( a, b, a );	// false
```

Esse utilitário `onlyOne(..)` apenas deve retornar `true` se exatamente um dos argumentos for `true` / verdadeiro. Ele está usando coerção *implícita* nas validações verdadeiras e coerção *explícita* nas outras, incluindo o valor final retornado.

Mas e se precisamos que esse utilitário seja capaz de gerenciar quatro, cinco ou vinte flags da mesma forma? É bem difícil imaginar implementar um código que seja capaz de gerenciar todas essas permutações de comparações.

Mas aqui está onde fazer a coerção de valores `boolean` para `number`s (`0` ou `1`, obviamente) pode ajudar muito:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		// pula os valores falsos. mesmo que tratar
		// eles como 0's, mas evita os NaN's.
		if (arguments[i]) {
			sum += arguments[i];
		}
	}
	return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a );				// true
onlyOne( b, a, b, b, b );		// true

onlyOne( b, b );				// false
onlyOne( b, a, b, b, b, a );	// false
```

**Obeservação:** Claro, em vez do loop `for` em `onlyOne(..)`, você pode usar a tarefa do ES5 `reduce(..)`, mas eu não queria obscurecer os conceitos.

O que estamos fazendo aqui é relacionado com coerção de `1` para `true`/verdadeiro, e adionando todos numericamente. `sum += arguments[i]` usa coerção *implícita* para fazer isso acontecer. Se um e apenas um valor na lista de `arguments` é `true`, então a soma numérica vai ser `1`, do contrário a soma não será `1` e portanto a condição desejada não será atendida.

Nós podemos claro fazer isso com coerção *implícita* no lugar:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += Number( !!arguments[i] );
	}
	return sum === 1;
}
```

Nós primeiro usamos `!!arguments[i]` para forçar a coerção dos valores para `true` ou `false`. Só assim você poderia passar os valores `boolean`, como `onlyOne( "42", 0 )`, e isso ainda continuará funcionando como esperado (do contrário você vai terminar com uma concatenação `string` e a lógica será incorreta).

Uma vez que temos certeza que é um `boolean`, nós fazemos outra coerção *explícita* com `Number(..)` para ter certeza que os valores são `0` ou `1`.

As formas de coerção *explícita* desse utilitário são "melhores"? Ela evita o `NaN` como explicado nos comentários do código. Mas, ultimamente, isso depende da sua necessidade. Eu pessoalmente acho que a forma anterior, confiando em coerção *implícita* é mais elegante (se você não tiver passando `undefined` ou `NaN`), e a versão *explícita* é desnecessariamente mais verbosa.

Mas assim como tudo o que discutimos aqui, é uma escolha.

**Observação:** Independentemente de abordagem *implícita* ou *explícita*, você pode facilmente fazer variações `onlyTwo(..)` ou `onlyFive(..)` simplismente mudando a comparação final de `1`, para `2` ou `5`, respectivamente. Isso é drasticamente mais fácil do que adicionar um monte de expressões `&&` e `||`. Então, geralmente, coerção é muito útil nesse caso.

### Implicitamente: * --> Boolean

Agora, vamos voltar nossa atenção para coerção *implícita* de valores `boolean`, como isso é de longe o mais comum e também de longe o mais potencialmente problemático.

Lembre-se, coerção *implícita* é o que entra quando você usa um valor de tal forma que ele força o valor a ser convertido. Para operações numéricas e de `string`, é bem fácil de ver como as coerções podem acontecer.

Mas, que tipo de expressões de operação requerem/forçam (*implicitamente*) uma coerção `boolean`?

1. A expressão test em uma declaração `if(..)`.
2. A expressão test (segunda cláusula) em um header `for ( .. ; .. ; .. )`.
3. A expressão test em loops `while (..)` e `do..while(..)`.
4. A expressão test (primeira cláusula) em expressões ternárias `? :`.
5. O operando *left-hand* (que serve uma expressão test -- veja abaixo!) para os operadores `||` ("lógico OU") e `&&` ("lógico E").

Qualquer valor usado nesse contexto que já não seja um `boolean` vai sofrer coerção *implícita* para um `boolean` usando as regras da operação abstrata `ToBoolean` abordada anteriormente nesse capítulo.

Vamos ver alguns exemplos:

```js
var a = 42;
var b = "abc";
var c;
var d = null;

if (a) {
	console.log( "yep" );		// yep
}

while (c) {
	console.log( "nope, never runs" );
}

c = d ? a : b;
c;								// "abc"

if ((a && d) || c) {
	console.log( "yep" );		// yep
}
```

Em todos estes contextos, os valores não `boolean`s sofrem coerção *implícita* para seus equivalentes `boolean` para fazer decisões de teste.

### Operadores `||` e `&&`

É bem provável que você já tenha visto os operadores `||` ("lógico OU") e `&&` ("lógico E") na maioria ou em todas outras linguagens que você já usou. Então seria natural presumir que eles trabalham basicamente da mesma forma no JavaScript como nas outras linguagens similares.

Há aqui uma nuance pouco conhecida, mas muito importante.

Na verdade, eu argumentaria que esses operadores nem sequer deveriam ser chamados de "operadores___lógicos", pois esse nome é incompleto ao descrever o que eles fazem. Se eu fosse dar à eles um nome mais preciso (se mais desajeitado), eu os chamaria de "operadores de seletores", ou mais completo, "operadores de seletor de operandos".

Por quê? Porque eles, na verdade, não resultam em um valor *lógico* (também conhecido como `boolean`) no JavaScript, como eles fazem em algumas outras linguagens.

Então qual o *resultado* deles? Eles retornam o valor de um (e apenas um) de seus dois operandos. Em outras palavras, **eles selecionam um dos dois valores de operandos**.

Citação da seção 11.11 da especificação ES5:

> O valor produzido pelo operador && ou || não pe necessariamente do tipo Boolean. O valor prodizido sempre será o valor de uma das duas empressões de operandos.

Vamos ilustrar:

```js
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

**Espera, o quê?** Pense nisso. Em liunguagens como C e PHP, essas expressões resultam em `true` ou `false`, mas em JS (e Python e Ruby, aliás!), o resultado vem dos próprios valores.

Ambos operadores, `||` e `&&` fazem um teste `boolean` no **primeiro operando** (`a` ou `c`). Se o operando já não for um `boolean` (que no caso, não é), uma coerção `ToBoolean` normal acontece, então o teste pode ser feito.

Para o operador `||`, se o teste é `true`, a expressão `||` resulta no valor do *primeiro operando* (`a` ou `c`). Se o teste é `false`, a expressão `||` resulta no valor do *segundo operando* (`b`).

Inversamente, para o operador `&&`, se o teste é `true`, a expressão `&&` resulta no valor do *segundo operando* (`b`) . Se o teste é `false`, a expressão `&&` resulta no valor do *primeiro operando* (`a` ou `c`).

O resultado das expressões `||` ou `&&` é sempre o valor de um dos operandos, **não** o resultado (possivelmente convertido) do teste. Em `c && b`, `c` é `null`, e portanto falso. Mas a própria expressão `&&` resulta em `null` (o valor em `c`), não no `false` convertido usado no teste.

Viu como esses operaodres agem como "seletores de operandos" agora?

Outra forma de pensar sobre esses operadores:

```js
a || b;
// aproximadamente equivalente à:
a ? a : b;

a && b;
// aproximadamente equivalente à:
a ? b : a;
```

**Observação:** Eu chamo `a || b` de "aproximadamente equivalente" à `a ? a : b` porque a saída é idêntica, mas há uma diferença de nuance. Em `a ? a : b`, se `a` era uma expressão mais complexa (como por exemplo uma que pode ter efeitos colaterais como chamar uma `function`, etc..), então a expressão `a` vai possivelmente ser avaliada duas vezes (se a primeira avaliação for verdadeira). Por contraste, para `a || b`, a expressão `a` é avalidada apenas uma vez, e esse valor é usado tanto para o teste coercivo como para o valor do resultado (se apropriado). A mesma nuance se aplica para as expressões `a && b` and `a ? b : a`.

Um uso extremamente comum e útil desse comportamento, em que há uma grande chance de você já ter usado isso antes e não entendido completamente é:

```js
function foo(a,b) {
	a = a || "hello";
	b = b || "world";

	console.log( a + " " + b );
}

foo();					// "hello world"
foo( "yeah", "yeah!" );	// "yeah yeah!"
```

O idioma `a = a || "hello"` (às vezes se diz que é a versão do JavaScript do "null coalescing operator" do C#) testa `a` e se ele não tem valor (ou apenas um valor falso indesejável), provê um valor padrão de backup (`"hello"`).

No entanto, **tenha cuidado!**

```js
foo( "That's it!", "" ); // "That's it! world" <-- Opa!
```

Viu o problema? `""` como segundo argumento é uma valor falso (veja `ToBoolean` anteriormente nesse capítulo), então o teste `b = b || "world"` falha, e o valor padrão `"world"` é substituído, mesmo quando a intenção era, provavelmente, passar explicitamente que `""` seja o valor atribuído para `b`.

Essa linguagem `||` é extremamente comum, e bem útil, mas você tem que usá-la somente em casos onde *todos os valores falsos* devem ser ignorados. Do contrário, você precisará ser mais explícito no seu teste, e provalmente usar um ternário `? :` no lugar.

Essa *atribuição de valor padrão* é tão comum (e útil!) que até mesmo aqueles que veementemente e publicamente condenam a coerção JavaScript, freuqnetemente a utilizam em seu pŕoprio código!

E o `&&`?

Esse é outra linguagem que é bem menos comum, mas que é usada por minificadores JS frequentemente. O operador `&&` "seleciona" o segundo operando se, e apenas se, o primeiro teste  do operando for verdadeiro, e esse uso é chamado algumas vezes de "operador guarda" (veja também "Curto-circuito" no capítulo 5) -- o primeiro teste de expressão "guarda" a segunda expressão:

```js
function foo() {
	console.log( a );
}

var a = 42;

a && foo(); // 42
```

`foo()` é chamada apenas porque o teste de `a` é verdadeiro. Se esse teste falha, essa declaração de expressão `a && foo()` vai apenas parar silenciosamente -- isso é conhecido como "curto-circuito" -- e nunca chamar `foo()`.

De novo, não é muito comum que as pessoas criem essas coisas. Normalmente, elas fazem `if (a) { foo(); }` no lugar. Mas os minificadores JS escolhem `a && foo()` porque é muito mais curto. Então, agora, se você alguma vez tiver que decifrar tal código, você saberá o que ele está fazendo e por quê.

Ok, então `||` e `&&` têm alguns truques na manga, com tanto que você queira permitir a coerção *implícita* nessa mistura.

**Observação:** Ambos, `a = b || "something"` e `a && b()` referem-se ao comportamento de circuitos curtos, que nos abordamos com mais detalhes no capítulo 5.

O fato desses operadores. na verdade, não resultarem em `true` e `false` possivelmente mexerá um pouco com a sua cabeça agora. Você provavelmente está se perguntando como todos suas declarações `if` e seus loops `for` funcionavam, se eles incluíram expressões lógicas compostas como `a && (b || c)`.

Não se preocupe! o céu não está desabando. Seu código está (provavelmente) bem. É que você provavelmente nunca percebeu antes que havia uma coerção *implícita* para `boolean` acontecendo **depois** que a expresão composta era analisada.

Considere:

```js
var a = 42;
var b = null;
var c = "foo";

if (a && (b || c)) {
	console.log( "yep" );
}
```

Esse código continua funcionando da forma que você sempre achou que funcionava, exceto por um detalhe sutil. A expressão `a && (b || c)` *na verdade* resulta em `"foo"`, não `true`. Portanto, a declaração `if` *então* força o valor `"foo"` a sofrer coerção para `boolean`, o que é claro será `true`.

Viu? não há razão para entrar em pânico. Seu código, provavelmente, está à salvo. Mas agora você sabe mais sobre como isso faz o que faz.

E agora você também percebeu que tal código usa coerção *implícita*. Se você ainda está no time "evite coerção (implícita)", você terá que voltar e fazer todos aqueles testes *explicitamente*:

```js
if (!!a && (!!b || !!c)) {
	console.log( "yep" );
}
```

Boa sorte com isso! ... Desculpe, apenas provocando.

### Coerção de Symbols

Até esse ponto, não houve quase nenhuma diferença de resultado observável entre coerção *explícita* e *implícita* -- apenas a legibilidade do código está em jogo.

Mas os Symbols do ES6 introduzem uma pegadinha no sistema de coerção que nós precisamos discutir brevemente. Por razões que vão bem além do escopo do que nós vamos discutir nesse livro, coerção *explícita* de um `symbol` para uma `string` é permitida, mas coerção *implícita* do mesmo não é permitida e lançará um erro.

Considere:

```js
var s1 = Symbol( "cool" );
String( s1 );					// "Symbol(cool)"

var s2 = Symbol( "not cool" );
s2 + "";						// TypeError
```

Valores `symbol` não fazem coerção para `number` de nenhuma forma (lança um erro de qualquer jeito), mas estranhamente ambas podem fazer coerção *explícita* e *implícita* para `boolean` (sempre `true`).

Consistências são sempre fáceis de aprender, e exceções nunca são divertidas de lidar, mas nós apenas precisamos ter cuidado com os novos valores `symbol` do ES6 e como nós fazemos coerção nelas.

A boa notícia: provavelmente será extremamente raro você precisar fazer coerção de uma valor `symbol`. A maneira como eles são normalmente usados (veja o capítulo 3), provavelmente não exigirá coerção em uma base normal.

## Igualdade Ampla vs. Igualdade Estrita

Igualdade ampla é o operador `==`, e igualdade estrita é o operador `===`. Ambos operadores são usados para comparar dois valores para "igualdade", mas o "amplo" vs. "estrito" indica uma diferença de comportamento **muito importante** entre os dois, especificamente em como eles decidem a "igualdade".

Um equívico muito comum sobre esses dois operadores é: `==` verifica igualdade de valores e `===` verifica igualdade de ambos, valores e tipos. Enquanto isso parece sensato, é impreciso. Incontáveis livros bem respeitados de JavaScript e blogs disseram exatamente isso, mas infelizmente eles estão todos *errados*.

A descrição correta é: "`==` permite coerção na comparação da igualdade e `===` não permite."

### Desempenho da Igualdade

Pare e pense sobre a diferença entre a primeira explicação (imprecisa) e esta segunda (precisa).

Na primeira explicação, parece óbvio que `===` está *fazendo mais trabalho* que `==`, porque ele precisa *também* verificar o tipo. Na segunda explicação, `==` é o que está *fazendo mais trabalho* porque ele precisa seguir através dos passos da coerção se os tipos são diferentes.

Não caia na armadilha, como muitos fazem, de pensar que isso tem alguma coisa a ver com performance, como se `==` fosse ser mais lento que `===` de qualquer maneira relevante. Embora seja mensurável que a coerção tome *um pouco mais* de tempo de processamento, são meros microsegundos (sim, isso é um milionésimo de segundo!).

Se você está comprando dois valores do mesmo tipo, `==` e `===` usam o algoritmo idêntico, e algumas outras diferenças mínimas na implementação do motor, eles devem fazer o mesmo trabalho.

Se você está comparando dois valores de tipos diferentes, a performance não é o fator importante. O que você deveria se perguntar é: ao comparar esses dois valores, eu quero a coerção ou não?

Se você quer a coerção, use `==` igualdade ampla, mas se você não quer coerção, use `===` igualdade estrita.

**Observação:** a implicação aqui é que ambos `==` e `===` verifiquem os tipos dos seus operandos. A diferença é em como eles irão responder se os tipos não coincidem.

### Igualdade abstrata

O comportamento do operador `==` é definido como "O algoritmo de comparação de igualdade abstrata" na seção 11.9.3 da especificação ES5. O que está listado lá é um algoritmo abrangente, mas simples, que declara explicitamente todas as combinações possíveis de tipos, e como as coerções (se necessárias) devem acontecer para cada combinação.

**Atenção:** Quando (*implicitamente*) a coerção é vista como sendo muito complicada e também defeituosa para ser uma *boa parte útil*, são essas regras de "igualdade abstrata" que estão sendo condenadas. Geralmente, elas são ditas como muito complexas e não intuitivas para desenvolvedores aprenderem e usá-las na prática, e que elas mais causam bugs nos programas JS do que provêm grande legibilidade do código. Eu acredito que essa é uma premissa defeituosa -- que vocês leitores são desenvolvedores competentes que escrevem (e leêm e entendem!) algoritmos (códigos) durante todo o dia. Então o que se segue é uma plano de exposição das "igualdades abstratas" em termos simples. Mas eu imploro para que você também leia a seção 11.9.3 da especificação ES5. Eu acho qe você ficará surpreso do quão sensata ela é.

Basicamente, a primeira cláusula (11.9.3.1) diz, se os dois valores que estão sendo comparados são do mesmo tipo, eles são simplesmente e naturalmente comparados via identidade como você esperava. Por exemplo, `42` só é igual a `42`, e `"abc"` é apenas igual à `"abc"`.

Algumas pequenas exceções à expectativa normal para estar ciente são:

* `NaN` nunca é igual a ela mesma (veja Capítulo 2)
* `+0` e `-0` são iguais entre si (veja Capítulo 2)

A última provisão na cláusula 11.9.3.1 é para comparação de igualdade ampla `==` com `object`s (incluindo `function`s e `array`s). Tais valores são apenas *iguais* se ambos referenciam para *exatamente o mesmo valor*. Não ocorre coerção aqui.

**Observação:** A comparação de igualdade estrita `===` é definida identicamente para 11.9.3.1, incluindo a provisão sobre dois valores de `objects`. É um fato pouco conhecido que **`==` e `===` se comportam de forma idêntica** no caso inde dois `objects`s estão sendo comparados.

O resto do algoritmo em 11.9.3. especifica qur se você usar igualdade ampla `==` para comparar dois valores de tipos diferentes, um ou ambos os valores precisarão sofrer coerção *implícita*. Essa coerção acontece para que ambos valores eventualmente terminem com o mesmo tipo, no qual possam ser comparados pela igualdade usando valores de identidade simples.

**Observação:** A operação de não-igualdade ampla `!=` é definida exatamente como você esperava, na medida em que é literalmente a comparação da operação `==` realizada na sua totalidade, e então a negação do resultado. O mesmo vale para a operação de não-igualdade estrita `!==`.

#### Comparando: `string`s com `number`s

Para ilustrar a coerção `==`, vamos primeiro desconstruir os exemplos anteriores `string` e `number` desse capítulo:

```js
var a = 42;
var b = "42";

a === b;	// false
a == b;		// true
```

Como esperávamos, `a===b` falha, porque nenhuma coerção é permitida, e de fato, os valores `42` e `"42"` são diferentes.

No entanto, a segunda comparação `a == b` usa igualdade ampla, que significa que se acontecer dos tipos serem diferentes, a comparação do algoritmo vai fazer uma coerção *implícita* em um ou ambos os valores.

Mas qual, exatamente, é o tipo de coerção que acontece aqui? O valor `a` de `42` torna-se uma `string`, ou o valor `b` de `"42"` torna-se um `number`?

Na cláusula 11.9.3.4-5 da especificação ES5 diz:

> 4. Se o Type(x) é um Number e o Type(y) é uma String,
>    retorna o resultado da comparação x == ToNumber(y).
> 5. Se o Type(x) é uma String e Type(y) ié um Number,
>    retorna o resultado da comparação ToNumber(x) == y.

**Atenção:** A especificação usa `Number` e `String` como nomes formais para os tipos, enquanto esse livro prefere `number` e `string` para tipos primitivos. Não deixe a capitalização de `Number` na especificação te confundir com a função nativa `Number()`. Para nossos propósitos, a capitalização do nome do tipo é irrelevante -- eles têm, basicamente, o mesmo significado.

Claramente, a especificação diz que o valor `"42"` sofre coerção para um `number` na comparação. O *como* dessa coerção já foi abordada anteriormente, especificamente com a operação abstrata `ToNumber`. Nesse caso, é bem óbvio que os dois valores `42` resultantes são iguais.

#### Comparando: qualquer coisa com `boolean`

Uma das maiores pegadinhas com a coerção *implícita* da igualdade ampla `==` aparecem quando você tenta comparar um valor diretamente com `true` ou `false`.

Considere:

```js
var a = "42";
var b = true;

a == b;	// falso
```

Espera, o que aconteceu aqui? Nós sabemos que `"42"` é um valor verdadeiro/*thruthy* (veja anteriormente neste capítulo). Então, como ele não é `==`, igualdade ampla à `true`?

A razão é simples e enganosamente complicada. É tão fácil de cometer um equívico, muitos desenvolvedores JS nunca prestam atenção suficiente para compreendê-la.

Vamos citar novamente a especificação, cláusula 11.9.3.6-7:

> 6. Se Type(x) é Boolean,
>    retorna o valor da comparação ToNumber(x) == y.
> 7. Se Type(y) é Boolean,
>    retorna o valor da comparação x == ToNumber(y).

Vamos acabar com isso. Primeiro:

```js
var x = true;
var y = "42";

x == y; // false
```

O `Type(x)` é de fato `Boolean`, então ele executa `ToNumber(x)`, que faz a coerção de `true` para `1`. Agora `1 == "42"` é avaliado. Os tipos continuam diferentes, então (essencialmente recursivamente) nós reconsultamos o algoritmo, que assim como acima, irá fazer a coerção de `"42"` para `42`, e `1 == 42` é claramente `false`.

Reverta isso, e nós teremos a mesma saída:

```js
var x = "42";
var y = false;

x == y; // false
```

O `Type(y)` é `Boolean` dessa vez, então `ToNumber(y)` custa `0`. `"42" == 0` recursivamente torna-se `42 == 0`, que claro, é `false`.

Em outras palavras, **o valor `"42"` não é nem `== true` nem `== false`.** Primeiramente, essa declaração pode parecer loucura. Como pode um valor não ser nem verdadeiro nem falso?

Mas esse é o problema! Você está fazendo a pergunta, totalmente errada. Não é sua culpa, de verdade. Seu cérebro está te enganando.

`"42"` é de fato verdadeiro/*truthy*, mas `"42" == true` **não está executando um teste/coerção de boolean** de maneira nenhuma, não importa o que seu cérebro diga. `"42"` não está sofrendo coerção para um `boolean` (`true`), mas, em vez disso, `true` é que está sofrendo coerção para `1`, e então `"42"` estão sofrendo coerção para `42`.

Quer nos agrade ou não, `ToBoolean` nem está envolvido aqui, então a verdade ou falsidade de `"42"` é irrelevante para a operação `==`!

O que *é* relevante, é entender como o algoritmo de comparação `==` se comporta com todas as diferentes combinações. No que se refere à um valor `boolean` de qualquer lado do `==`, um `boolean` sempre sofre coerção para um `number` *primeiro*.

Se isso parece estranho para você, você não está sozinho. Eu pessoalmente recomendaria à nunca, nunca mesmo, em nenhuma circunstância, usar `== true` ou `== false`. Nunca.

Mas lembre-se, eu estou falando somente do `==` aqui. `=== true` e `=== false` não permitirá a coerção, então você está seguro dessa coerção `ToNumber` oculta.

Considere:

```js
var a = "42";

// péssimo (vai falhar!):
if (a == true) {
	// ..
}

// ruim também (vai falhar!):
if (a === true) {
	// ..
}

// bom o bastante (funciona implicitamente):
if (a) {
	// ..
}

// melhor (funciona explicitamente):
if (!!a) {
	// ..
}

// ótimo também (funciona explicitamente):
if (Boolean( a )) {
	// ..
}
```

Se você sempre evita usar `== true` ou `== false` (também conhecido como igualdade ampla com `boolean`s) no seu código, você nunca terá que se preocupar sobre essa pegadinha mental de verdadeiro/falso.

#### Comparando: `null`s com `undefined`s

Outro exemplo de coerção *implícita* pode ser visto com `==` igualdade ampla entre valores `null` e `undefined`. Citando de novo a especificação ES5, cláusula 11.9.3.2-3:

> 2. Se x é null e y é undefined, retorna true.
> 3. Se x é undefined e y é null, retorna true.

`null` e `undefined` quando comparados com `==` igualdade ampla, equipararm-se (fazem coerção) uns nos outros (assim como neles próprios, obviamente), e nenhum outro valor em toda a linguagem.

Isso significa que `null` e `undefined` podem ser tratados sem distinção para propósitos de comparação, se você usar o operador `==` igualdade ampla para permitir coerção *implícita* mútua.

```js
var a = null;
var b;

a == b;		// true
a == null;	// true
b == null;	// true

a == false;	// false
b == false;	// false
a == "";	// false
b == "";	// false
a == 0;		// false
b == 0;		// false
```

A coerção entre `null` e `undefined` é segura e previsível e nenhum outro valor pode dar falsos positivos em tal teste. Eu recomendo usar essa coerção para permitir que `null` e `undefined` sejam indistinguíveis e assim tratados como o mesmo valor.

Por exemplo:

```js
var a = doSomething();

if (a == null) {
	// ..
}
```

A verificação `a == null` vai passar somente se `doSomething()` retornar ambos, `null` ou `undefined`, e vai falhar com qualquer outro valor, mesmo outro valor falso como `0`, `false` e `""`.

A forma de verificação *explícita*, que não permite nenhum tipo de coerção, é (eu acho) desnecessariamente muito mais feia (e talvez um pouco menos performática!):

```js
var a = doSomething();

if (a === undefined || a === null) {
	// ..
}
```

Na minha opinião, a forma `a == null` é ainda outro exemplo de onde a coerção *implícita* melhora a legibilidade do código, mas faz isso de uma maneira confiável e segura.

#### Comparando: `object`s com não-`object`s

Se um `object`/`function`/`array` é comparado com um escalar primitivo simples (`string`, `number` ou `boolean`), a especificação ES5 diz na cláusula 11.9.3.8-9:

> 8. Se Type(x) é tanto uma String ou um Number e Type(y) é um Object,
>    retorna o resultado da comparação x == ToPrimitive(y).
> 9. Se Type(x) é um Object e Type(y) tanto uma String ou um Number,
>    retorna o resultado da comparação ToPrimitive(x) == y.

**Observação:** Você pode notar que essas cláusulas apenas mencionam `String` e `Number`, mas não `Boolean`. Isso porque, como dito antes, a clásula 11.9.3.6-7 trata da coerção de qualquer operando `Boolean` apresentado para um `Number` primeiro.

Considere:

```js
var a = 42;
var b = [ 42 ];

a == b;	// true
```

O valor `[42]` tem sua operação abstrata `ToPrimitive` chamada (veja a seção anterior "Valores de operações abstratas"), que resulta no valor `"42"`. Daqui em diante, é apenas `42 == "42"`, que como já abordamos, torna-se `42 == 42`, então `a` e `b` são coercitivamente iguais.

**Dica:** Todos os quirks da operação abstrata `ToPimitive` que nós discutimos anteriormente nesse capítulo (`toString()`, `valueOf()`) são aplicados aqui como nós esperávamos. Isso pode ser bem útil se você tiver uma estrutura de dados complexa que você quer definir um método personalizado em `valueOf()`, para fornecer uma valor simples para propósitos de comparação de igualdade.

No capítulo 3, nós abordamos "unboxing", onde um `object` wrapper em torno de uma valor primitivo (como de `new String("abc")`, por exemplo) é desencapsulado, e o valor primitivo subjacente ("abc") é retornado. Esse comportamento está relacionado à coerção `ToPrimitive` no algoritmo `==` :

```js
var a = "abc";
var b = Object( a );	// Mesmo que `new String( a )`

a === b;				// false
a == b;					// true
```
`a == b` é `true` porque `b` sofre coerção (ou "unboxed", desencapsulado) via `ToPrimitive` para seu valor escalar primitivo "abc" subjacente, que é o mesmo que o valor em `a`.

Há alguns valores onde isso não é o caso, por conta de outras regras primárias do algoritmo de `==`. Considere:

```js
var a = null;
var b = Object( a );	// Mesmo que `Object()`
a == b;					// false

var c = undefined;
var d = Object( c );	// Mesmo que `Object()`
c == d;					// false

var e = NaN;
var f = Object( e );	// Mesmo que `new Number( e )`
e == f;					// false
```

Os valores `null` e `undefined` não podem ser encapsulados (*boxed*) -- eles não tem um object wrapper equivalente -- Então o `Object(null)` é como o `Object()` em que ambos apenas produzem um objeto normal.

`NaN` pode ser encapsulado no seu object wrapper `Number` equivalente, mas quando `==` causa uma desencapsulamento, a comparação `NaN == NaN` falha porque `NaN` nunca é igual a si mesmo (veja o capítulo 2).

### Casos à parte

Agora que nós examinamos completamente como a coerção *implícita* de `==` igualdade ampla funciona (tanto na maneira sensível como na surpreendente), vamos tentar chamar os piores e mais loucos casos para que possamos ver o que precisamos evitar para não ser pego com bugs de coerção.

Primeiro, vamos examinar como modificar prototypes nativos podem produzir resultados loucos:

#### Um número por outro valor seria...

```js
Number.prototype.valueOf = function() {
	return 3;
};

new Number( 2 ) == 3;	// true
```

**Atenção:** `2 == 3` não teria caído nessa armadilha, porque nem `2` nem `3` teria invocado o método nativo `Number.prototype.valueOf()` porque ambos já são valores primitivos `number` e podem ser comparados diretamente. No entanto, `new Number(2)` deve passar pela coerção `ToPrimitive`, e por isso invocar `valueOf()`.

Maldade né? É claro que é. Ninguém nunca deveria fazer algo assim. O fato de que você *pode* fazer isso é usado como crítica da coerção e `==`. Mas isso é uma frustação mal direcionada. JavaScript não é *ruim* por que você pode fazer tais coisas, um desenvolvedor é *ruim* **se eles fizerem tais coisas**. Não caia na falácia "minha linguagem de programação deveria me proteger de mim mesmo".

Próximo vamos considerar outro exemplo complicado, o que leva a maldade do exemplo anterior para outro nível:

```js
if (a == 2 && a == 3) {
	// ..
}
```

Você pode pensar que isso seria impossível, porque `a` nunca deveria ser igual a ambos `2` e `3` *ao mesmo tempo*. Mas "ao mesmo tempo" é impreciso, já que a primeira expressão `a == 2`, acontece estritamente *antes* de `a == 3`.

Então, e se nós fizermos com que `a.valueOf()` tivesse efeitos colaterais toda vez que fosse chamado, de modo que na primeira vez retorna `2` e na segunda vez que for chamada retorne `3`? Muito fácil:

```js
var i = 2;

Number.prototype.valueOf = function() {
	return i++;
};

var a = new Number( 42 );

if (a == 2 && a == 3) {
	console.log( "Yep, this happened." );
}
```

De novo, esses são truques maldosos. Não faça-os. Mas também não os use como queixas contra a coerção. Abusos potenciais dos mecanismos não são evidências suficientes para condenar o mecanismo. Apenas evite esses truques malucos, e mantenha-se com o uso válido e apropriado da coerção.

#### Comparações False-y

A queixa mais comum contra coerção *implícita* na comparação `==` ver de quão surpreendentes os valores *falsy* se comportam quando comparados entre si.

Para ilustrar, vamos olhar para a lista de casos à parte sobre comparação de valores *falsy*, para ver quais são os razoáveis e os problemáticos:

```js
"0" == null;			// false
"0" == undefined;		// false
"0" == false;			// true -- UH OH!
"0" == NaN;				// false
"0" == 0;				// true
"0" == "";				// false

false == null;			// false
false == undefined;		// false
false == NaN;			// false
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
false == {};			// false

"" == null;				// false
"" == undefined;		// false
"" == NaN;				// false
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
"" == {};				// false

0 == null;				// false
0 == undefined;			// false
0 == NaN;				// false
0 == [];				// true -- UH OH!
0 == {};				// false
```

Nessa lista de 24 comparações, 17 delas são bem razoáveis e previsíveis. Por exemplo, nós sabemos que `""` e `NaN` não são valores iguais, e mesmo eles não sofrem coerção para serem igualdades amplas, considerando que `"0"` e `0` são razoavelmente igualáveis e *vão* sofrer coerção como igualdade ampla.

No entanto, sete destas comparaçãoes estão marcadas com "UH OH!" porque como falsos positivos, elas mais provavelmente são pegadinhas que podem te enganar. `""` e `0` são valores definitivamente distintos, e é raro que você queira tratá-los como iguais, então a coerção mútua é problemática. Note que não há nenhum falso negativo aqui.

#### Os Loucos 

No entanto, nós não temos que parar aqui. Nós podemos continuar procurando por mais coerções problemáticas:

```js
[] == ![];		// true
```

Oooo, isso parece estar em um nível mais alto de loucura, certo!? Seu cérebro pode estar te enganando que você está comparando um valor verdadeiro com falso, então o resultado `true` é surpreendente, como nós *sabemos*, um valor nunca pode ser verdadeiro e falso ao mesmo tempo!

Mas, na verdade, não é isso que está acontecendo. Vamos destrinchar isso. O que sabemos sobre o operador unário `!`? Ele aplica a coerção explícita para um `boolean` usando as regras de `ToBoolean` (e também troca a paridade). Então antes de `[] == ![]` sequer ser processado, ele já traduziu para `[] == false`. Nós já vimos essa forma em nossa lista acima (`false == []`), então seu resultado surpreendente *não é novo* para nós.

E sobre esses outros casos?

```js
2 == [2];		// true
"" == [null];	// true
```

Como nós dissemos anteriormente em nossa discussão `ToNumber`, o lado direito dos valores `[2]` e `[null]` vão passar pela coerção `ToPrimitive` e então eles podem ser comparados mais prontamente aos primitivos simples (`2` e `""`, respectivamente) no lado esquerdo. Desde que `valueOf()` para o próprio valor `array`, a coerção falha na stringficação do `array`.

`[2]` torna-se `"2"`, o que então sofre coerção `ToNumber` para `2` para o valor do lado direito na primeira comparação. `[null]` apenas continua sendo `""`.

Então, `2 == 2` e `"" ==""` são completamente compreensíveis.

Se seu instinto é continuar desgostando destes resultados, sua frustação, na verdade, não é com a coerção, como provavelmente você pensa que é. É na verdade uma queixa contra o comportamento padrão de valores `array` `ToPrimitive` de uma coerção de `[2]` e então `"2"`, exceto talvez `"[2]"` -- mais isso pode ser muito estranho em outros contextos!

Você poderia justificar que, desde que `String(null)` torna-se `"null"`, então `String([null])` deverá também tornar-se `"null"`. Essa é uma afirmação razoável. Então, esse é o verdadeiro culpado.

Coerção *implícita* por si só não é a vilã aqui. Até mesmo uma coerção *explícita* de `[null]` para uma `string` resulta em `""`. O que está em contradição é se é sensato para um valor `array` stringficar para um equivalente de seu conteúdo, e exatamente como isso acontece. Então, direcione sua frustação para as regras de `String( [..] )`, porque é de onde a loucura vem. Talvez não deva mesmo acontecer a stringficação de um `array`? Mas isso teria muitas outras desvantagens em outras partes da linguagem.

Outra pegadinha famosa citada:

```js
0 == "\n";		// true
```

Como discutimos antes com `""`, `"\n"` (ou `" "` ou qualquer outra combinação de espaços vazios) sofre coerção via `ToNumber`, e o resultado é `0`. Que outro valor de `number` você espera que um espaço vazio seja convertido? Te encomoda que `Number(" ")` retorne `0`?

Realmente o único outro `number` razoável no qual strings vazias ou espaços em brancos possam sofrer coerção é o `NaN`. Mas isso seria *realmente* melhor? A comparação `" " == NaN` vai certamente falhar, mas não está claro que teríamos realmente *corrigido* qualquer uma das preocupações subjacentes.

As chances de que um programa JS real falhe porque `0 == "\n"` são terrivelmente raras, e tais casos podem facilmente ser evitados.

Conversões de tipo **sempre** tem casos à parte, em qualquer linguagem -- nada especificamente para coerção. Os problemas aqui são sobre adivinhar um certo conjunto de casos à parte (e, talvez, corretamente!), mas esse não pe um argumento saliente contra o mecanismo geral de coerção.

Quase qualquer coerção louca entre *valores normais* que você provavelmente irá encontrar (além de hacks intencionalmente complicados `valueOf()` or `toString()` como anteriores) se resumirão a esta lista curta de sete coerções que nós identificamos acima.

Para contrastar contra estes 24 suspeitos prováveis para pegadinhas de coerção, considere outra lista como esta:

```js
42 == "43";							// false
"foo" == 42;						// false
"true" == true;						// false

42 == "42";							// true
"foo" == [ "foo" ];					// true
```

Nesses casos não falsos, não à parte (e há literalmente um número infinito de comparações que podemos colocar nesta lista), os resultados da coerção são totalmente seguros, razoáveis ​​e explicáveis.

#### Teste de Sanidade

OK, nós definitivamente achamos algumas coisas loucas quando nós olhamos a fundo na coerção *implícita*. Não é à toa que a maioria dos desenvolvedores afirmam que coerção é ruim e deve ser evitada, certo!?

Mas vamos voltar um passo e fazer um teste de sanidade.

Para fins de comparações de magnitude, nós temos *uma lista* de sete pegadinhas de coerção problemáticas, mas nós temos *outra lista* de (ao menos 17, mas atualmente infinita) coerções que são totalmente sensatas e explicáveis.

Se você está buscando por um exemplo de texto para "matar uma mosca com um canhão", é isto: descartando a totalidade da coerção (a infinitamente larga lista de comportamentos seguros e úteis) por causa de uma lista de, literalmente, sete pegadinhas.

A reação mais prudente seria perguntar, "como eu posso usar incontáveis *partes boas* da coerção, mas evitar as poucas *partes ruins*?

Vamos dar uma olhada novamente na lista *ruim*:

```js
"0" == false;			// true -- UH OH!
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

Quatro dos sete itens dessa lista envolvem a comparação `== false`, que nós dissemos anteriormente que você deve **sempre**, **sempre** evitar. Essa é uma regra bem fácil de lembrar.

Agora a lista caiu para três.

```js
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

São essas coerções razoáveis que você faria em um programa normal de JavaScript? Em quais condições elas realmente aconteceriam?

Eu não acho que é absurdamente provável que você use `== []` em um teste `boolean` no seu programa, ao menos não se você sabe o que está fazendo. Você propavelmente faria `== ""` ou `== 0` no lugar, como:

```js
function doSomething(a) {
	if (a == "") {
		// ..
	}
}
```

Você teria um Oops se você acidentalmente chamasse `doSomething(0)` ou `doSomething([])`. 
Outro cenário:

```js
function doSomething(a,b) {
	if (a == b) {
		// ..
	}
}
```

Novamente, isso pode quebrar se você fizesse algo como `doSomething("",0)` ou `doSomething([],"")`.


Então, embora *possam* existir situações em que essas coerções vão te pegar, e você provavelmente vai querer ter cuidado com elas, elas provavelmente não são super comuns em toda sua base de código.

#### Usando coerção implícita com segurança

O conselho mais importante que posso te dar: examine seu programa e razões sobre quais valores podem aparecer em ambos lados de uma comparação `==`. Para efetivamente evitar problemas com tais comparações, aqui estão algumas heurísticas para seguir:

1. Se ambos lados de uma comparação pode ter valores `true` ou `false`, nunca, NUNCA use `==`.

2. Se ambos lados da comparação pode ter valores `[]`, `""`, ou `0`, considere seriamente em não usar `==`.

Nesses cenários é quase sempre melhor usar `===` em vez de `==`, para evitar coerções indesejadas. Siga essas duas regras simples e basicamente todas as pegadinhas de coerção que poderiam te afetar serão efetivamente evitadas.

**Ser mais explícito/verboso nesses casos irá te salvar de muitas dores de cabeça.**

A questão de `==` vs. `===` é apropriadamente enquadrada como: você deve permitir coerção para uma comparação ou não?

Há muitos casos que tal coerção pode ser útil, permitindo que você expresse mais tersamente alguma lógica de comparação (como com `null` e `undefined`, por exemplo).

No geral, há relativamente poucos casos onde coerção *implícita* é verdadeiramente perigosa. Mas nesses lugares, por segurança, definitivamente use `===`

**Dica:** Outro lugar onde é garatido que a coerção não te prejudique é com o operador `typeof`. `typeof` sempre irá te retornar uma de sete strings (veja o Capítulo 1), e nenhuma delas serão strings vazias `""`. Sendo assim, não há nenhum caso onde checar o tipo de algum valor irá executar uma coerção *implícita*. `typeof x == "function"` é 100% tão seguro quanto `typeof x === "function"`. Literalmente, a especificação diz que o algoritmo será idêntico nesse caso. Então, não use `===` cegamente em todo lugar porque aquilo é o que as ferramentas do seu código dizem para fazer, ou (pior ainda) porque você viu em algum livro para **não pensar nisso**. A qualidade do seu código é sua.

A coerção *implícita* é maligna e perigosa? Em alguns casos, sim, mas geralmente, não.

Seja um desenvolvedor maduro e responsável. Aprenda como usar o poder da coerção (tanto *explícita* como *implícita*) efetivamente e com segurança. E ensine aqueles à sua volta a fazer o mesmo.

Aqui está uma tabela útil feita pelo Alex Dorey (@dorey no GitHub) para visualizar uma variedade de conversões:

<img src="fig1.png" width="600">

Source: https://github.com/dorey/JavaScript-Equality-Table

## Comparação Relacional Abstrata

Enquanto essa parte da coerção implícita geralmente recebe bem menos atenção, é importante pensar no que acontece com comparações `a < b` (similar à `a == b` que já examinamos em profundidade).

O algoritmo da "Comparação relacional abstrata" na seção 11.8.5 do ES5 essencialmente se divide em duas partes: o que fazer se a comparação envolve ambos valores `string` (segunda metade), ou qualquer outra coisa (primeira metade).

**Observação:** O algoritmo é apenas definido por  `a < b`. Então, `a > b` é manipulado como `b < a`.

O algoritmo primeiro chama a coerção `ToPrimitive` de ambos os valores, e se o resultado de qualquer uma das chamadas não for uma `string`, então ambos valores são convertidos para valores `number` usando as regras de operação `ToNumber`, e comparado numericamente.

Por exemplo:

```js
var a = [ 42 ];
var b = [ "43" ];

a < b;	// true
b < a;	// false
```

**Observação:** As ressalvas semelhantes para `-0` e `NaN` aplicam-se aqui como feitas no algoritmo `==` discutido anteriormente.

Entretanto, se ambos os valores são `string` para a comparação `<`, a comparação lexográfica simples (alfabético natural) é performada nos caracteres:

```js
var a = [ "42" ];
var b = [ "043" ];

a < b;	// false
```

`a` e `b` não são convertidos para `number`, porque ambos terminam como `string` depois da conversão `ToPrimitive` nos dois `array`s. Então, `"42"` é comparado caractere por caractere com `"043"`, começando com os primeiros caracteres `"4"` e `"0"`, respectivamente. Como `"0"` é lexicograficamente *menor que* `"4"`, a comparação retorna `false`.

Exatamente o mesmo comportamento e objetivo acontece para:

```js
var a = [ 4, 2 ];
var b = [ 0, 4, 3 ];

a < b;	// false
```

Aqui, `a` torna-se `"4,2"` e `b` torna-se `"0,4,3"`, e esses se comparam lexicograficamente de forma idêntica ao trecho anterior.

E sobre:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// ??
```

`a < b` também é `false`, porque `a` torna-se `[object Object]` e `b` torna-se `[object Object]`, e então claramente `a` não é lexograficamente menor que `b`.

Mas estranhamente:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false
a > b;	// false

a <= b;	// true
a >= b;	// true
```

Por que `a == b` não é `true`? Eles são o mesmo valor de `string` (`"[object Object]"`), então parece que eles deveriam ser iguais, certo? Não. Relembre a discussão anterior sobre como `==` funciona com referêcias de `object`.

Mas então como `a <= b` e `a >= b` resultam em `true`, se `a < b` **e** `a == b` **e** `a > b` são todos `false`?

Porque a especificação diz que para `a <= b`, ele vai na verdade avaliar primeiro `b < a`, e então negar esse resultado. Desde que `b < a` seja *também* `false`, o resultado de `a <= b` é `true`.

Isso provavelmente é muito contrário de como você teria explicado o que o `<=` faz até agora, o que provavelmente teria sido o literal: "menor que *ou* igual a." O JS mais precisamente considera `<=` como "não é maior que" (`!(a > b)`, que o JS trata como `!(b < a)`). Além disso, `a >= b` é explicado primeiro considerando-o como `b <= a`, e então aplicando o mesmo reciocícnio.

Infelizmente, não existe "comparação relacional estrita" como é para a igualdade. Em outras palavras, não há como prevenir coerção *implícita* de ocorrer com comparações relacionais como `a < b`, além de garantir que `a` e `b` são explicitamente do mesmo tipo antes de ser feita a comparação.

Use o mesmo raciocínio para nossa discussão anterior sobre teste de sanidade `==` vs. `===`. Se a coerção é útil e razoavelmente segura, como em uma comparação de `42 < "43"`, **use-a**. Por outro lado, se você precisa ter certeza sobre uma comparação relacional, faça a *coerção explícita* dos valores primeiro, antes de usar `<` (ou suas contrapartes).

```js
var a = [ 42 ];
var b = "043";

a < b;						// false -- comparação de string!
Number( a ) < Number( b );	// true -- comparação de number!
```

## Revisão

Nesse capítulo, nós voltamos nossa atenção para como as conversões de tipos acontecem no JavaScript, chamadas **coerção**, na qual pode ser caracterizada como *explícita* ou *implícita*.

Coerção tem uma má reputação, mas ela é na verdade bastante útil em muitos casos. Uma tarefa importante para um desenvolvedor JavaScript responsável é tirar um tempo para aprender todas as entradas e saídas da coerção para decidir quais partes irão ajudar a melhorar seu código, e quais partes ele realmente deve evitar.

Coerção *explícita* é o código que é óbvio que a intenção é converter um valor de um tipo para outro. O benefício é a melhora na legibilidade e manutenabilidade do código reduzindo a confusão.

Coerção *implícita* é a coerção que está "escondida" como um efeito colateral de alguma outra operação, onde não é tão óbvio o tipo de conversão que vai acontecer. Enquanto parece que a coerção *implícita* é o oposto da *explícita*, e portanto, é ruim (e de fato, muitos pensam que sim!), na verdade, coerção *implícita* é também sobre melhorar a legibilidade do código.

Especialmente para *implícita*, coerção deve ser usada com responsabilidade e conscientemente. Saber por que você está escrevendo o código que está escrevendo, e como ele funciona. Esforce-se para escrever códigos que outros facilmente possam aprender e entender também.
