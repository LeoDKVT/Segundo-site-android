# You Don't Know JS: Tipos e Gramática
# Capitulo 4: Coerção

Agora que nós entendemos melhor os tipos e valores do JavaScript, vamos voltar nossa atenção para um tópico muito controverso: coerção.

Como mencionado no Capítulo 1, os debates sobre se coerção é um recurso útil ou uma falha no design da linguagem (ou algo no meio disso!) têm ocorrido desde o primeiro dia. Se você já leu outros livros sobre JS, você sabe que a *mensagem* prevalentemente esmagadora é que coerção é mágica, má, confusa, e francamente uma ideia ruim.

Seguindo o mesmo espírito dessa série de livros, ao invés de fugir da coerção porque todo mundo faz isso, ou porque você tem algum capricho, eu acho que você deve seguir em frente e procurar entender mais claramente.

Nosso objetivo é entender plenamente os prós e contras (sim, *existem* prós!) da coerção, então você poderá estar bem informado para tomar a decisão de como ela se adequa ao seu programa.

## Convertendo Valores

Converter um valor de um tipo para outro é geralmente chamado de "type casting", quando explicitamente, e "coerção" quando feito implicitamente (forçado pelas regras de como um valor é utilizado).

**Nota:** Pode não ser óbvio, mas as coerções do JavaScript sempre resultam em um dos primitivos escalares (veja Capítulo 2), como `string`, `number`, ou `boolean`. Não existe coerção que resulte em um valor complexo como `object` ou `function`. O Capítulo 3 aborda "boxing", que envolve os valores primitivos em seus `object` homólogos, mas isso não é realmente coerção em seu sentido correto.

Pode-se distinguir esses de ainda outra maneira: "type casting" (ou coversão de tipos) ocorrem em tempo de compilação em linguagem staticamente tipadas, enquanto "coerção" é uma conversão que ocorre em tempo de execução em linguagens dinamicamente tipadas.

Entretanto, em JavaScript, a maioria das pessoas chamam todos esse tipos de *coerção*, por isso a maneira que prefiro chamar de "coerção implícita" vs. "coerção explícita".

A diferença deve ser óbvia: "coerção explícita" é quando consegue-se olhar para o código e ver que a coversão de tipos está ocorrendo intencionalmente, enquanto que a "coerção implícita" é quando o conversão de tipos ocorre de maneira óbvia como um efeito colateral de outra operação intencional.

Por exemplo, considere essas duas abordagens de coerção:

```js
var a = 42;

var b = a + "";			// coerção implícita

var c = String( a );	// coerção explícita
```

Para `b`, a coerção ocorre implicitamente, porque o operador `+` combinado com um dos operandos sendo uma `string` (`""`) vai insistir que a operação seja uma concatenação de strings (juntando duas strings), que, *como um efeito colateral (oculto)* vai forçar o valor `42` em `a` a ser convertido a seu equivalente em `string`: `"42"`.

Em contrapartida, a função `String(..)` torna bastante óbvio que o valor de `a` é convertido em sua representação em `string`.

As duas abordagens têm o mesmo efeito: `42` vira `"42"`. Mas é o *como* que é o coração dos debates mais acalorados sobre coerção no JavaScript.

**Note:** Tecnicamente, há pequenas nuances no comportamento que vão além da estética. Nós abordaremos isso em mais detalhes mais tarde neste capítulo, na seção "Implicitamente: Strings <--> Number".

Os termos "explícito" e "implícito", ou "óbvio" e "efeito colateral oculto", são *relativos*.

Se você sabe exatamente o que `a + ""` está fazendo e você está intencionalmente convertendo o valor para uma `string`, você pode achar a operção suficientemente "explícita". Por outro lado, se você nunca viu a função `String(..)` usada para coerção de strings, o seu comportamento pode parecer oculto o suficiente para ser "implícito" para você.

Mas nós estamos discutindo "explícito" vs. "implícito" baseados nas prováveis opiniões de um desenvolvedor *mediano, razoavelmente informado, mas não especialista ou devoto pela especificação do JS*. Se você se encaixa de alguma forma nesse grupo, você terá de ajustar a sua perspectiva para estar de acordo com as nossas observações.

Relembrando: é pouco provável que após escrevermos o nosso código, nós sejamos os únicos que vão lê-lo. Mesmo que você saiba todos os prós e contras no JS, considere como um colega de trabalho com menos experiência vai ser se sentir quando ler o seu código. Vai ser "explícito" ou "implícito" para eles da mesma maneira que é para você?

## Operações de valor abstrato

Antes de explorarmos a coerção *explícita* e *implícita*, nós precisamos aprender as regras básicas que governam como os valores *tornam-se* uma `string`, `number` ou `boolean`. A seção 9 da especificação ES5 define várias "operações abstratas" (nome técnico para "operação interna") com as regras de conversão de valor. Nós vamos prestar atenção, especificamente, em: `ToString`, `ToNumber` e `ToBoolean`, e menos na extensão  `ToPrimitive`.

### `ToString`

Quando um valor não-`string` é convertido para uma representação `string`, a conversão é manipulada pela operação abstrata `ToString` na seção 9.8 da especificação.

Valores primitivos nativos têm stringficação natural: `null` torna-se `"null"`, `undefined` torna-se `"undefined"` e `true` torna-se `"true"`. `numbers` são geralmente expressos de forma natural como você esperava, mas como discutimos no Capítulo 2, `numbers` muito pequenos ou muito grandes são representados na forma expoente:

```js
// multiplicando `1.07` por `1000`, sete vezes mais
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;

// sete vezes três dígitos => 21 digits
a.toString(); // "1.07e21"
```

Para objetos regulares, a menos que você mesmo especifique, o padrão `toString()` (localizado em Object.prototype.toString()`) vai retornar uma *`[[Class]]` interna* (veja o capítulo 3), como por exemplo `"[object Object]"`.

Mas como mostrado anteriormente, se um objeto tem seu próprio método `toString()`, e se você usa esse objeto em um tipo `string`, o `toString()` é o que será chamado automaticamente, e o resultado da `string` dessa chamada é o que vai ser usado no lugar.

**Observação:** A forma que um objeto é convertido em uma `string` tecnicamente passa através da operação abstrata `toPrimitive` (seção 9.1 da especificação ES5), mas essas nuances serão abordadas com mais detalhes na seção `ToNumber`, mais tarde nesse capítulo, então vamos pulá-lo aqui.

Arrays têm um padrão `toString()` substituído que stringfica a (string) concatenação de todos esses valores (cada um stringficando a si mesmo), com `","` entre cada valor:

```js
var a = [1,2,3];

a.toString(); // "1,2,3"
```

Novamente, `toString()` pode tanto ser chamada explicitamente, ou ela vai ser chamada automaticamente se uma não-`string` for usada em um contexto de `string`.

#### Stringficação do JSON

Outra tarefa que parece terrível relacionada a `ToString` é quando você usa a funcionalidade  `JSON.stringify(..)` para serializar um valor para um valor `string` compatível com JSON.

É importante notar que essa stringficação não é exatamente a mesma que a coerção. Mas como ela é relacionada às regras de `ToString` acima, nós faremos uma leve diversificação para abordar os comportamentos de stringficação JSON aqui.

Por mais simples que sejam o valores, a stringficação JSON se comporta basicamente da mesma forma que conversões `toString()`, exceto que a serialização resulta *sempre como uma `string`*:

```js
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (uma string com um valor dentro de aspas)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

Qualuqer valor *seguro para JSON* pode ser stringficada com `JSON.stringify(..)`. Mas o que é *seguro para JSON (JSON-safe)* ? Qualquer valor que pode ser representado em uma representação JSON válida.

Pode ser mais fácil considerar valores que **não** são seguros para JSON. Alguns exemplos: `undefined`s, `function`s, (ES6+) `symbol`s, e `object`s com referências circulares (onde as referências de propriedade em uma estrutura de objeto criam um ciclo interminável entre si). Todos esses são valores ilegais para uma estrutura JSON padrão, principalmente porque ela não têm portabilidade para outras linguagens que consumem valores JSON.

A funcionalidade `JSON.stringify(..)` vai omitir automaticamente valores `undefined`, `function` e `symbol` quando cruzar com eles. Se o valor em questão for encontrado em um `array`, esse valor é substituído por `null` (então aquela posição da informação do array não é alterada). Se for encontrado como propriedade de um objeto, essa propriedade vai simplesmente ser excluída.

Considere:

```js
JSON.stringify( undefined );					// undefined
JSON.stringify( function(){} );					// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );		// "{"a":2}"
```

Mas se você tentar fazer um `JSON.stringify(..)` em um `object` com referência(s) circular nele, um erro vai ser lançado.

Stringficação JSON tem um comportamente especial, que se o valor de um `object` tem um método `toJSON()` definido, esse método vai ser chamado primeiro para pegar um valor a ser usado para serialização.

Você tem a intenção de stringficar um objeto JSON que pode conter valores JSON ilegais, ou se você apenas têm, valores no `object` que não são apropriados para a serialização, você deveria definir um método `toJSON()`para que ele retorne à uma versão *segura para JSON* do `object`.

Por exemplo:

```js
var o = { };

var a = {
	b: 42,
	c: o,
	d: function(){}
};

// cria uma referência circular dentro de `a`
o.e = a;

// vai lançar um erro na referência circular
// JSON.stringify( a );

// define um valor de serialização JSON personalizado
a.toJSON = function() {
	// apenas inclui a propriedade `b` para serialização
	return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

É bem comum o equívoco que `toJSON()` deveria retornar uma representação stringficada de JSON. Isso está provavelmente incorreto, a menos que você queira realmente stringficar a própria `string` (geralmente não!). `toJSON()` deve retornar o valor regular atual (de qualquer tipo) seria apropriado, e o próprio `JSON.stringify(..)` vai manipular a stringficação.

Em outras palavras, `toJSON()` deve ser interpretado como "adequado para stringficação para um valor seguro para JSON", não "para uma string JSON" como muitos desenvolvedores assumem errôneamente.

Considere:

```js
var a = {
	val: [1,2,3],

	// provavelmente correto!
	toJSON: function(){
		return this.val.slice( 1 );
	}
};

var b = {
	val: [1,2,3],

	// provavelmente incorreto!
	toJSON: function(){
		return "[" +
			this.val.slice( 1 ).join() +
		"]";
	}
};

JSON.stringify( a ); // "[2,3]"

JSON.stringify( b ); // ""[2,3]""
```

Na segunda chamada, nós stringficamos o retorno `string` ao invés do próprio `array`, o que provavelmente não é o que queríamos fazer.

Enquanto estamos falando de `JSON.stringify(..)`, vamos discutor algumas funcionalidades pouco conhecidas que continuam a ser bem úteis.

Um segundo argumento opcional pode ser passado para `JSON.stringify(..)` que é chamado *substituto (replacer)*. Esse argumento pode tanto ser um `array` ou uma `function`. É usado para personalizar a serialização recursiva de um `object` fornecendo um mecanismo de filtro no qual propriedades podem ou não serem incluídas, em uma maneira similar de como o `toJSON()` pode preparar um valor para serialização.

Se um *substituto* é um `array`, ele deve ser um `array` de `strings`, no qual cada um vai especificar uma nome de propriedade que é permitida para ser incluída na serialização do `object`. Se uma propriedade que existe não está nessa lista, ela será ignorada.

Se o *substituto* é uma `function`, ele será chamado uma vez pelo próprio `object`, e então uma vez para cada propriedade no `object`, e cada vez que ele passar dois argumentos, *chave* e *valor*. Para ignorar uma *chave* na serialização, retorne `undefined`. Do contrário, retorne o *valor* fornecido.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

**Observação:** No caso do *substituto* da `function`, o argumento chave `k` é `undefined` na primeira chamada (onde o próprio objeto `a` está sendo passado). A declaração `if` **filtra** a propriedade nomeada de `"c"`. Stringficação é recursiva, então o array `[1,2,3]` tem cada um dos seus valores (`1`, `2`, e `3`) passados como `v` para o *substituto*, com índices (`0`, `1`, and `2`) como `k`.

Um terceiro argumento opcional também pode ser passado para `JSON.stringify(..)`, chamado *espaço (space)*, no qual é usado como indentação para deixar a saída mais bonita e amigável. *espaço* pode ser um intermediador positivo para indicar quantos espaços de caracteres devem ser usados em cada nível de identação. Ou, *espaço* pode ser uma `string`, que nesse caso até os primeiros dez caracteres do seu valor serão usados para cada nível de identação.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

Lembre-se, `JSON.stringify(..)` não é uma forma direta de coerção. Nós o abordamos aqui, porém, por duas razões que seu comportamento está relacionado com coerção `ToString`:

1. Valores `string`, `number`, `boolean`, e `null` todos podem ser stringficados para JSON basicamente o mesmo como a forma que eles convertem valores `string` através das regras da operação abstrata `ToString`.
2. Se você passou o uma valor de `object` para `JSON.stringify(..)`, e esse `object` tem um método `toJSON()` nele, `toJSON()` é chamado automaticamente para (tipo que) "converter" o valor para ser *seguro para JSON* antes da stringficação.

### `ToNumber`

Se qualquer valor não-`number` é usado de uma forma que exige que seja um `number`, como uma operação matemática, a especificação ES5 define, na seção 9.3, a operação abstrata `ToNumber`.

Por exemplo, `true` torna-se `1` e `false` torna-se `0`. `undefined` torna-se `NaN`, mas (curiosamente) `null` torna-se `0`.

`ToNumber` para valor `string` essencialmente funciona para a maioria das partes como regras/sintaxe para numéricos literais (Veja o Capítulo 3). Se isso falhar, o resultado é `NaN` (ao invés de um erro de sintaxe com `numbers` literais). Um exemplo da diferença é que `0`-números octais pré fixados não são manipulados como octais (apenas como decimais normais) nessa operação, portanto esses octais são válidos como `numbers` literais (veja o Capítulo 2).

**Observação:** As diferenças entre a gramática de `number` literal  e `ToNumber` em um valor de uma `string` são sutis e altamente matizados, e por isso não serão mais abordados aqui. Consulte a seção 9.3.1 da especificação ES5 para mais informações.

Objetos (e arrays) vão primeiro ser convertidos para seus valores primitivos equivalentes, e o valor resultado (se for primitivo mas ainda não um `number`) é convertido para um `number` de acordo com as regras de `ToNumber` mencionadas.

Para converter para seu valor primitivo equivalente, a operação abstrata`ToPrimitive` (seção 9.1 da especificação ES5) irá consultar o valor (usando a operação interna `DefaultValue` -- seção 8.12.8 da especificação ES5) em questão para ver se ele tem um método `valueOf()`. Se o `valueOf()` estiver disponível e ele retornar um valor primitivo, *aquele* valor é usado para coerção. Do contrário, mas se `toString()` está disponível, ele vai fornecer o valor para a coerção.

Se nenhuma das operações pode fornecer um valor primitivo, um `TypeError` é lançado.

A partir de ES5, você pode criar certos objetos não coercivos -- um sem `valueOf()` e `toString()` -- se ele tiver um valor `null` para seu `[[Prototype]]`, geralmente criado com `Object.create(null)`. Veja o título *this & Object Prototypes* desa série para mais informações de `[[Prototype]]`s.

**Observação:** Nós abordamos como converter para `number`s em detalhes mais tarde nesse capítulo, mas para esse próximo trecho de código, apenas suponha que a função `Number(..)` faz isso.

Considere:

```js
var a = {
	valueOf: function(){
		return "42";
	}
};

var b = {
	toString: function(){
		return "42";
	}
};

var c = [4,2];
c.toString = function(){
	return this.join( "" );	// "42"
};

Number( a );			// 42
Number( b );			// 42
Number( c );			// 42
Number( "" );			// 0
Number( [] );			// 0
Number( [ "abc" ] );	// NaN
```

### `ToBoolean`

A seguir, vamos ter uma pequena conversa sobre como `boolean`s se comportam em JS. Há **muita confusão e equívoco** em torno desse tópico, então preste bastante atenção!

Em primeiro lugar, JS tem as palavras-chave atuais `true` e `false`, e elas se comportam exatamente como você esperaria de valores `boolean`. É um equívoco comum que os valores `1` e `0` sejam idênticos à `true/false`. Enquanto isso pode ser verdadeiro em outras linguagens, em JS os `number`s são `number`s e os `boolean`s são `boolean`s. Você pode converter `1` para `true` (e vice-versa) ou `0` para `false` (e vice versa). Mas eles não são os mesmos.

#### Valores Falsos (Falsy)

Mas esse não é o fim da história. Nós precisamos discutir como outros valores além dos dois `boolean`s se comportam independentemente de você converter *para* seus equivalentes `boolean`.

Todos os valores JavaScript podem ser divididos em duas categorias:

1. Valores que irão se tornar `false` se convertidos para `boolean`
2. Todo o resto (o que vai obviamente se tornar `true`)

Eu não estou apenas sendo engraçado. A especificação JS define uma específica e estreita lista dos valores que poderão tornar-se `false` quando convertidos para um valor `boolean`.

Como sabemos qual é essa lista de valores? Na seção 9.2 da especificação ES5, é definido uma operação abstrata `ToBoolean`, na que diz exatamente o que aconteceria para todos os valores possíveis quando você tenta converte-los "para boolean".

A partir dessa tabela, obtemos o seguinte da chamada lista de valores "falsos":

* `undefined`
* `null`
* `false`
* `+0`, `-0`, e `NaN`
* `""`

É isso. Se um valor não está nessa lista, é um valor "falso" (falsy), e ele não vai ser convertido para `false` se você forçar uma coerção `boolean` nele.

Por conclusão lógica, se um valor *não* está nessa lista, ele deve estar em *outra lista*, na qual nós chamamos de lista de valores "verdadeiros". Mas o JS realmente não define uma lista de valores "verdadeiros" por si só. Ele dá alguns exemplos, assim como dizemos explicitamente que todos os objetos são verdadeiros, mas principalmente a especificação apenas implica que: **qualquer coisa que não esteja explicitamente na lista falsa, é portanto, verdadeira.**

#### Objetos Falsos (Falsy Objects)

Espere um minuto, aquele títulos de seção soa até contraditório. Eu *apenas disse* literalmente que a especificação chama todos os objetos de verdadeiro, certo? Não deveria existir tal coisa como um "objeto falso".

O que isso possivelmente pode significar?

Você deve estar tentado a pensar que isso significa um *object wrapper* (veja o capítulo 3) em torno de um valor falso (como `""`, `0` ou `false`). Mas não caia nessa *armadilha*.

**Observação** 

Wait a minute, that section title even sounds contradictory. I literally *just said* the spec calls all objects truthy, right? There should be no such thing as a "falsy object."

What could that possibly even mean?

You might be tempted to think it means an object wrapper (see Chapter 3) around a falsy value (such as `""`, `0` or `false`). But don't fall into that *trap*.

Essa é uma piada de especificação sutil que alguns de vocês podem ter sacado.

Considere:

```js
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
```

Nós sabemos que todos os três valores são *objects wraper* (veja o capítulo 3) em torno de valores obviamente falsos. Mas esses objetos se comportam como `true` ou como `false`? Essa é fácil de responder:

```js
var d = Boolean( a && b && c );

d; // true
```

Então, todos os três se comportam como `true`, como essa é a única maneira de `d` acabar como `true`.

**Dica:** Note que o wrapped `Boolean(..)` está em torno da expressão `a && b && c` -- você deve estar se perguntando porque isso está ali. Nós vamos voltar mais tarde nesse capítulo, então faça um nota mental disso. Para um pequeno exercício, procure por si mesmo o que `d` será se você apenas fizer `d = a && b && c` sem a chamada `Boolean(..)`!

Então, se "objetos falsos" **não são apenas objetos embrulhados em torno de valores falsos**, o que diabos eles são?

A parte complicada é que eles podem aparecer no seu programa JS, mas eles na verdade não são parte do próprio JavaScript.

**O quê?!**

Há certos casos em que navegadores criam seus próprios tipos de comportamentos *exóticos* de valores, nomeando essa ideia de "objetos falsos", no topo da semântica regular do JS.

Um "objeto falso" é um valor que parece e age como um objeto normal (propriedades, etc.), mas quando você converte eles para um `boolean`, ele faz a coerção para um valor `false`.

**Por quê?!**

O caso mais conhecido é `document.all`: um tipo array (objeto) fornecido pelo seu programa JS *pelo DOM* (não pelo próprio motor JS), que expõe elementos na sua página para seu programa JS. Ele *costuma* se comportar como um objeto normal -- isso seria verdadeiro. Mas não mais.

O próprio `document.all` nunca foi realmente "padrão" e há muito tempo ficou obsoleto/abandonado.

"Eles não podem apenas remover isso então?" Desculpe, boa tentativa. Gostaria que pudessem. Mas há muita base de código JS legado por aí que dependem desse uso.

Então, porque fazer ele agir como falso? Porque coerções de `document.all` para `boolean` (assim como nas declarações `if`) foram quase sempre usadas como um meio de detectar o IE antigo e não padronizado.

O IE há muito tempo vem se aproximando dos padrões e, em muitos casos, vem empurrando a Web para frente tanto ou mais do que qualquer outro navegador. Mas todos aqueles códigos `if` antigos (document.all){ /* it's IE */ }` antigos contiuam por aí, e muito deles, provavelmente, nunca irão embora. Todos esses códigos legados continuam supondo que estão sendo executados em IE antigos, o que leva a más experiências de navegação para usuários IE.

Então, nós não podemos remover `document.all` completamente, mas o IE não quer que códigos `if (document.all) { .. }` funcionem mais, então esses usuários em IE modernos terão novas lógicas de código compatível com os padrões.

"O que devemos fazer?" **"Já sei! Vamos degradar o sistema de tipo do JS e fingir que `document.all` é falso!"

Eca. Isso é uma merda. É um macete louco que a maioria dos desenvolvedores não entendem. Mas a alternativa (fazer nada sobre os problemas sem solução acima) fede *um pouquinho mais*.

Então...é isso que temos: "objetos falsos" loucos e fora do padrão adicionados ao JS pelos navegadores. Ebaa!

#### Valores verdadeiros (truthy)

De volta para a lista verdadeira. O que exatamente são valores verdadeiros? Lembre-se: ** um valor é verdadeiro se ele não está na lista falsa **.

Considere:

```js
var a = "false";
var b = "0";
var c = "''";

var d = Boolean( a && b && c );

d;
```

Qual valor você espera que `d` tenha aqui? Deve ser ou `true` ou `false`.

É `true`. Porquê? Porque apesar dos conteúdos dos valores daquelas `string` parecerem como valores falsos, os próprios valores da `string` são verdadeiros, porque `""` é o único valor de `string` na lista falsa.

E esses?

```js
var a = [];				// array vazio -- verdadeiro ou falso?
var b = {};				// object vazio --  verdadeiro ou falso?
var c = function(){};	// function vazio --  verdadeiro ou falso?

var d = Boolean( a && b && c );

d;
```

Sim, você acertou, `d` continua `true` aqui. Porque? Mesma razão de antes. Apesar do que possa parecer, `[]`, `{}`, e `function(){}` *não* estão na lista falsa, e, portanto, são valores verdadeiros.

Em outras palavras, a lista de verdadeiros é infinitamente longa. É impossível de fazer tal lista. Você apenas pode fazer uma lista falsa e *cosultá-la*.

Pegue cinco minutos, escreva a lista falsa em um post-it para o monitor do seu computador, ou memorize-a se preferir. De qualquer forma, você facilmente poderá construir uma lista falsa virtual sempre que precisar simplesmente perguntando se está na lista falsa ou não.

A importância de verdadeiro e falso no entendimento de como um valor vai se comportar quando você coverte-lo (explicitamente ou implicitamente) para uma valor `boolean`. Agora que você tem essas duas listas em mente, nós podemos mergulhar nos exemplos de coerção.

## Coerção Explícita

Coerção *explícita* refere-se à conversões de tipos que são óbvias e explícitas. Há uma ampla variedade de uso de conversões de tipo que caeem na categoria de coerção *explícita* para a maioria dos desenvolvedores.

O objetivo aqui é indetificar padrões em seu código que nós podemos deixar claro e óbvio que nós estamos convertendo um valor de um tipo para o outro, para não deixar buracos para os futuros desenvolvedores tropeçarem. Quanto mais explícitos somos, mais provável que alguém mais tarde consiga ler nosso código e entender sem esforço excessivo qual era a nossa intenção.

Seria difícil encontrar quaisquer discordâncias salientes com coerção *explícita*, uma vez que se alinha mais de perto com o modo como a prática comumente aceita de conversão de tipos funciona em linguagens com tipagem estática. Como tal, vamos tomar como certo (por enquanto) que a coerção *explícita* pode ser aceita como não maligna ou controversa. No entando, nós vamos revisitar isso mais tarde.

### Explicitamente: Strings <--> Numbers

Nós vamos começar com a operação de coerção mais simples, e talvez, mais comum: converter valores entre uma representação `string` e `number`.

Para converter `string`s em `number`s,  nós usamos as funções nativas String(..)` e `Number(..)` (nas quais nos referimos no Capítulo 3 como "construtores nativos"), mas **muito importante**, nós não usamos a palavra-chave `new` na frente delas. Assim sendo, não estamos criando object wrappers.

Em vez disso, nós na verdade criamos uma *coerção explícita* entre os dois tipos:

```js
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
```

`String(..)` converte qualquer outro valor para um valor primitivo `string`, usando as regras da operação `ToString` discutida anteriormente. `Number(..)` coverte qualquer outro valor para um valor primitivo `number`, usando as regras da operação `ToNumber` discutida anteriormente.

Eu chamo isso de coerção *explícita* porque em geral, é bastante óbvio para a maioria dos desenvolvedores que o resultado final dessas operações é a aplicação da conversão de tipo.

Na verdade, a forma como isso é usado, realmente parece muito em como isso é feito em algumas outras linguagem de tipagem estática.

Por exemplo, em C/C++, você pode dizer tanto `(int)x` ou `int(x)`, que ambas irão converter o valor em `x` para um número inteiro. Ambas formas são válidas, mas muitos preferem a última, que meio que parece a chamda de uma função. No JavaScript, quando você diz `Number(x)`, isso parece terrivelmente igual. Importa se isso é *de verdade* uma chamada de função no JS? Na verdade, não.

Além de `String(..)` e `Number(..)`, há outras formas de converter "explicitamente" esses valores entre `string` e `number`:

```js
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

Chamar `a.toString()` é ostensivamente explícito (está bem claro que "toString" significa "para uma string"), mas existe alguma implicidade. `toString()` não pode ser chamado em um valor *primitivo* como `42`. Então o JS automaticamente "encaixota" (veja o Capítulo 3) `42` em um object wrapper, então `toString()` pode ser chamada contra o objeto. Em outras palavras, você pode chamar isso de "explicitamente implícito".

Aqui o `+c` esta mostrando o *operador unário* (operador com somente um operando) do operador `+`. Em vez de realizar uma adição matemática (ou concatenação de string -- veja abaixo), o unário `+` converte explicitamente seu operando (`c`) em um valor `number`.

`c+` é uma coerção explícita? Depende da sua experiêcnia e perspectiva. Se você sabe (e voce, agora, sabe!) que o unário `+` é explicitamente destinado para uma coerção `number`, então é bem explícito e óbvio. Porém, se você nunca viu isso antes, isso pode ser terrivelmente confuso, implícito, com efeitos colaterais ocultos, etc.

**Observação:** A perspectiva geralmente aceita na comunidade open source JS pe que o unário `+` é uma forma aceita de coerção *explícita*.

Mesmo se você realmente goste da forma `+c`, há, definitivamente, alguns lugares onde isso pode parecer terrivelmente confuso, Considere:

```js
var c = "3.14";
var d = 5+ +c;

d; // 8.14
```

O operador unário `-` também converte da mesma forma que o `+` faz, mas ele também inverte o sinal do número. No entanto, você não pode colocar dois `--` próximo um do outro para desvirar o sinal, como isso é feito com o operador de decremnto. Em vez disso, você vai precisar fazer: `- -"3.14"` com espaço no meio, e isso vai resultar na coerção para `3.14`.

Você provavelmente pode inventar todo o tipo de combinações horríveis de operadores binários (como `+` para adição) ao lado da forma  unária de um operador. Aqui está outro exemplo maluco:

```js
1 + - + + + - + 1;	// 2
```

Você deve considerar fortemente evitar corção unária com `+` (ou `-`) quando ela é imediatamentamenta adjacente de outro operador. Enquando o de cima funciona, é quase que universalmente considerado uma má ideia. Mesmo `d = +c` (ou `d =+ c` também!) pode muito facilmente ser confudido com `d += c`, o que é inteiramente diferente!

**Observação:** Outro lugar extremamente confuso para usar o unário `+` é adjacente à outro operador que pode ser, `++` operador de incremento e `--` operador de decremento. Por exemplo: `a +++b`, `a + ++b`, and `a + + +b`. Veja "Efeitos colaterais de expressões" no Capítulo 5 para mais sobre `++`.

Lembre-se, nós estamos tentando ser explícitos e **reduzir** a confusão, não torná-la muito pior!

#### `Date` para `number`

Outro uso comum do operador unário `+` é converter um objeto `Date` para um `number`, porque o resultado é um valor timestamp unix (milisegundos passados desde 1 de Janeiro 1970 1970 00:00:00 UTC) que é a representação da data/hora.

```js
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000
```

O uso mais comum dessa representação é pegar o tempo *atual* como uma timestamp, assim:

```js
var timestamp = +new Date();
```

**Nota:** Alguns desenvolvedores estão cientes do "truque" sintático peculiar no JavaScript, é que o `()` definido em uma chamada de construtor (uma função chamada com `new`) é *opcional* se não há argumentos para passar. Então você pode se deparar com a forma `var timestamp = +new Date;`. No entanto, nem todos os desenvolvedores concordam que omitir o `()` melhora a legibilidade, pois é uma exceção de sintaxe incomum que se aplica apenas a forma de chamada `new fn()` e não a forma de chamada tradicional `fn()`.

Mas coerção não é a única forma de pegar o timestamp de uma objeto `Date`. A abordagem de não coerção talvez seja preferível, já que é mais explícita:

```js
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```

Mas uma opção de não coerção *ainda mais* preferível é usar a função estática adicionada no ES5 `Date.now()`:

```js
var timestamp = Date.now();
```

E se você quiser fazer o pollyfill de `Date.now()` para navegadores antigos, é bem simples:

```js
if (!Date.now) {
	Date.now = function() {
		return +new Date();
	};
}
```

Eu recomendaria pular as formas de coerção relacionadas à datas. Use `Date.now()` para timestamps *atuais* e `new Date(..).getTime()` para pergar um timestamp para uma data/hora *não atual* específica que você precise.

#### O curioso caso do `~`

Um operador coercivo JS que é frequentemente negligenciado e geralmente muito confudido é o operador til `~` (também conhecido como "operador bit a bit NOT"). Muitos dos que até compreendem o que ele faz, vão muitas vezes continuar a evitá-lo. Mas se mantendo no espírito do nossa abordagem nesse livro e série, vamos cavar isso e descobrir se o `~` tem algo de útil para nos dar.

Na seção "inteiros de 32-bit (signed)" do Capítulo 2, nós abordamos como operadores bit a bit em JS são definidos apenas por operações de 32-bit, o que sifnifica que eles forçam seus operando a entrarem em conformidade com representações de valores 32-bit. As regras para como isso acontece são controladas pela operação abstrata `ToInt32` (Especificação ES5, seção 9.5).

`ToInt32` primeiro faz uma coerção para um `ToNumber`, o que significa que se o valor é `"123"`, ele vai primeiro se tornar `123` antes das regras de `ToInt32` serem aplicadas.

Enquanto não é *tecnicamente* uma coerção em si (desde que o type não mude!), o uso de operadores bit a bit (como `|` ou `~`) com um certo valor `number` especial produz um efeito coercivo que resulta em um valor `number` diferente.

Por exemplo, primeiro vamos considerar o `|` "operador bit a bit OU" usado de outra forma em um idioma não-op `0 | x`, que (como o capítulo 2 mostrou) essencialmente apenas faz a conversão `ToInt32`:

```js
0 | -0;			// 0
0 | NaN;		// 0
0 | Infinity;	// 0
0 | -Infinity;	// 0
```

Esse números especiais não são representações 32-bit (desde que eles venham do padrão 64-bit IEEE 754 -- veja o Capítulo 2), então `ToInt32` apenas especifica `0` como resultado para esses valores.

É discutível se `0 | __` é uma forma *explícita* dessa operação coerciva `ToInt32` ou se ela é mais *implícita*. Pela perspectiva da especificação, é inquestionavelmente *explícita*, mas se você não compreende operações bit a bit nesse nível, isso pode parecer uma mágica mais *implícita*. No entanto, de acordo com outras afirmações nesse capítulo, nós vamos chamá-la de *explícita*.

Então vamos voltar nossa atenção para o `~`. O operador `~` primeiro faz a "coerção" para um valor `number` de 32-bit , e então realiza uma negativa bit a bit (lançando a paridade de cada bit).

**Observação:** Isso é bem similar em como `!` não apenas faz a coerção de seus valores para `boolean` mas também lança sua paridade (veja a discussão dos "unários `!`" depois).

Mas...o quê!? Porquê nos importamos com bits sendo lançados? Isso é algo bem específico, algo com muitas nuances. É bem raro que os desenvolvedores JS precisem raciocinar sobre bits individuais.

Outra forma de pensar sobre a definição de `~` vem da ciência da computação/Matemática old-school: `~` realiza dois complementos. Ótimo, obrigado, isso está totalmente claro!

Vamos tentar de novo: `~x` é aproximadamente o mesmo que `-(x+1)`. Isso é estranho, mas um pouco mais fácil de racionalizar. Então:

```js
~42;	// -(42+1) ==> -43
```

Você provavelmente continua imaginando o que diabos é toda essa coisa com o `~`, ou porque isso realmente importa para uma discussão sobre coerção. Vamos chegar ao ponto rapidamente.

Considere `-(x+1)`. Qual é o único valor que você pode realizar essa operação no qual ele irá produzir um resultado `0` (ou, tecnicamente, `-0`)? `-1`. Em outras palavras, `~` usado com uma gama de valores `number` produzirá um valor falso (facilmente coercível para `false`) `0` para o valor de entrada `-1`, e, de outra forma, para qualquer outro valor verdadeiro.

Por que isso é relevante?

`-1` é comumente chamado de um "sentinel value", o que basicamente significa um valor no qual é dado um significado semântico arbitrário dentro do conjunto maior de valores do primeiro tipo (`number`s). A linguagem C usa o sentinel value `-1` para muitas funções que retornam valores `>=0` para "sucesso" e `-1` para "falha".

O JavaScript adotou esse precedente ao definir a operação `string` de `indexOf(..)`, que busca por uma substring e, se encontrada, retorna sua posição de índice inicial, ou `-1` se não encontrada.

É bem comum tentar usar `indexOf(..)` não apenas como uma operação para pegar a posição, mas como uma verificação `boolean` que verifica a presença/ausência de uma sibstring em outra `string`. Veja agora como como desenvolvedores realizam essas verificações:

```js
var a = "Hello World";

if (a.indexOf( "lo" ) >= 0) {	// true
	// encontrado!
}
if (a.indexOf( "lo" ) != -1) {	// true
	// encontrado!
}

if (a.indexOf( "ol" ) < 0) {	// true
	// não encontrado!
}
if (a.indexOf( "ol" ) == -1) {	// true
	// não encontrado!
}
```

Eu acho um pouco grosseiro olhar para `>= 0` ou `== -1`. É basicamente uma "abstração vazada", na medida em que está vazando o comportamento de implementação subjacente -- o uso da sentinela `-1` para "falha" -- no meu código. Eu preferiria esconder tal detahe.

E agora, finalmente, nós vemos porque `~` pode nos ajudar! Usar `~` com `indexOf()` realiza a "coerção" (na verdade apenas transforma) o valor **para ser um `boolean` apropriado para coerção**:

```js
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- verdadeiro!

if (~a.indexOf( "lo" )) {	// true
	// encontrado!
}

~a.indexOf( "ol" );			// 0    <-- falso!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// não encontrado!
}
```

`~` pega o valor retornado de `indexOf(..)` e o transforma: em caso de "falha" `-1` nós teremos o falso `0`, e todo outro valor é verdadeiro.

**Observação:** O pseudo-algoritmo `-(x+1)` para `~` implicaria que `~-1` é `-0`, mas na verdade ele produz `0` porque a operação subjacente é na verdade bit a bit, não matemática.

Tecnicamente, `if (~a.indexOf(..))` ainda está confiando na coerção *implícita* da resultante `0` para `false` ou diferente de zero para `true`. Mas no geral, `~` ainda me parece mais como um mecanismo de coerção *explícita*, desde que você saiba o que pretende fazer nessa linguagem.

Eu acho esse é um código mais limpo do que o desorganizado `>= 0` / `== -1`.

##### Truncando bits

Há mais um lugar que `~` pode aparecer em um código: alguns desenvolvedores usam o til duplo `~~` para truncar a parte decimal de um `number` (aplicar "coerção" para um número "inteiro"). É comum (embora erroneamente) dizer que este é o mesmo resultado que chamar `Math.floor(..)`.

Como `~~` funciona, é que o primeiro `~` aplica a coerção `ToInt32` e faz o lançamento do bit e, em seguida, o segundo` ~ `faz outro lançamento de bit a bit, folheando todos os bits de volta para o estado original. O resultado final é apenas a "coerção" `ToInt32` (também conhecida como truncamento).

**Observação:** O lançamento bit a bit duplo de `~~` é muito parecido com o comportamento de paridade da negativa dupla `!!`, explicada mais tarde na seção "Explicitamente: * --> Boolean.

Porém, `~~` precisa de algum cuidado/esclarecimento. Primeiro, ele apenas funciona dependente de valores em 32-bit. Mas, mais importante, ele não funciona da mesma forma em números negativos como o `Math.floor(..)` faz!

```js
Math.floor( -49.6 );	// -50
~~-49.6;				// -49
```

Definindo o `Math.floor(..)`, apesar das diferenças, `~~x` pode truncar para um inteiro (32-bit). Mas o `x | 0` também faz, e aparentemente com (ligeiramente) *menos esforço*.

Então, porque você escolheria `~~x` em vez de `x | 0`? Precedência de operador (veja o Capítulo 5):

```js
~~1E20 / 10;		// 166199296

1E20 | 0 / 10;		// 1661992960
(1E20 | 0) / 10;	// 166199296
```

Assim como todos os outros conselhos aqui, use `~` e `~~`` como mecanismos explícitos para "coerção" e transformação de valores somente se todos que lêem/escrevem o código em questão estão propriamente cientes de como esses operadores funcionam!

### Explicitamente: Parseando strings numéricas

Um resultado semelhante para coagir uma `string` para um` number` pode ser conseguido parseando um `number` de um conteúdo de caracteres de uma `string`. Há no entanto, diferenças distintas entre esse parseamento e a conversão de tipo que examinamos acima. 

Considere:

```js
var a = "42";
var b = "42px";

Number( a );	// 42
parseInt( a );	// 42

Number( b );	// NaN
parseInt( b );	// 42
```

Parsear um valor numérico de uma string é *tolerante* para caracteres não numéricos -- isso apenas para de parsear da esquerda para a direita quando encontrado -- enquanto a coerção é *não tolerante* e falha, resultando no valor 'NaN`.

Parseamento não deve ser visto como um substituto para coerção. Essas duas tarefas, mesmo similares, têm propósitos diferentes. Realize o parser de uma `string` como um `number` quando você não sabe/se importa com outros caracteres não numéricos que possam existir no lado direito. Fazer a coerção de um `string` (para um `number`) quando os únicos valores aceitáveis são numéricos e algo como "42px" deve ser rejeitado como um `number`.

**Dica:** `parseInt(..)` tem um irmão gêmeo, `parseFloat(..)`, que (como parece) tira um número de ponto flutuante de uma string.

Não esqueça que `parseInt(..)` opera em valores `string`. Não faz absolutamente nenhum sentido passar um valor `number` para `parseInt(..)`. Nem faria sentido passar nenhum outro tipo de valor, como `true`, `function(){..}` ou `[1,2,3]`.

Se você passar uma não `string`, o valor que você passar vai automaticamente sofrer coerção para uma `string` primeiro (veja "`ToString`" anteriormente), o que vai claramente ser um tipo de coerção *implícita* oculta. É realmente uma péssima ideia confiar em tal comportamento no seu programa, então nunca use `parseInt(..)` em um valor que não seja uma `string`.

Antes do ES5, outra pegadinha existia com `parseInt(..)`, a qual era fonte de muitos bugs de programas JS. Se você não passasse um segundo argumento para indicar qual base numérica (conhecida como radix) usar para interpretar o conteúdo numérico da `string`, `parseInt(..)` iria olhar para os caracteres iniciais e adivinhar.

Se os primeiros dois caracteres fossem `"0x"` ou `"0X"`, o palpite (por convenção) era que você queria interpretar a `string` como um `number` de base hexadecimal(base-16). Por outro lado, se o primeiro caractere fosse `"0"`, o palpite (novamente, por convenção) era que você queria interpretar a `string` como um `number` de base octal (base-8).

`string`s hexadecimais (com iniciais `0x` ou `0X`) não são extremamente fáceis de se misturar. Mas a adivinhação do número octal mostrou-se diabolicamente comum. Por exemplo:

```js
var hour = parseInt( selectedHour.value );
var minute = parseInt( selectedMinute.value );

console.log( "The time you selected was: " + hour + ":" + minute);
```

Parece inofensivo, certo? Tente selecionar `08` para hora e `09` para os minutos. Você vai ter `0:0`. Por quê? porque nem `8` nem `9` são caracteres válidos em octais base-8.

A correção pré-ES5 foi simples, mas muito fácil de esquecer: **sempre passar `10` como o segundo argumento**. Isso era totalmente seguro:

```js
var hour = parseInt( selectedHour.value, 10 );
var minute = parseInt( selectedMiniute.value, 10 );
```

A partir da ES5, `parseInt(..)` não adivinhava mais octais. A menos que você diga o contrário, ele supõe caracteres base-10 (ou base-16 para prefixos `"0"`). Isso é muito melhor. Apenas tenha cuidado se seu código tenha que rodar em ambientes pré-ES5, que nesse caso você ainda vai precisar passar `10` para o radix.

#### Parseando não-Strings

Um exemplo um pouco infame do comportamento do `parseInt(..)` é destacado em uma publicação com uma piada sarcástica alguns anos atrás, tirando sarro desse comportamento JS:

```js
parseInt( 1/0, 19 ); // 18
```
A afirmação pretensiosa (mas totalmente inválida) foi: "Se eu passar infinito e parsear um número inteiro disso, eu deveria recuperar o infinito, não 18." Certamente, JS deve estar louco por esse resultado, certo?

Embora este exemplo seja obviamente artificial e irreal, vamos entrar na loucura por um momento e examinar se JS realmente é tão louco.

Primeiramente, o pecado mais óbvio cometido aqui é passar uma não-`string` para `parseInt(..)`. Não, não, não. Faça isso e você estará pedindo por problemas. Mas mesmo se você fizer, o JS, educadamente, faz a coerção do que você passa em uma `string` que pode tentar parsear.

Alguns poderão argumentar que esse é um comportamento irracional, e que `parseInt(..)` deveria operar em um valor não-`string`. Isso deveria lançar um erro? Isso seria muito a cara do Java, francamente. Eu estremeço ao pensar que JS deveria começar a lançar erros em todo o lugar para que o `try..catch` seja necessário em quase todas as linhas.

Ele deveria retornar `NaN`? Talvez. Mas... que tal:

```js
parseInt( new String( "42") );
```

Isso deveria falhar também? É um valor não-`string`. Se você quer que o wrapper de objeto `String` seja desenpacotado para `"42"`, então é realmente tão incomum que o `42` se torne primeiro `"42"` para que `42` possa ser analisado de volta?

Eu argumentaria que essa coerção meio *explícita*, meio *implícita* que pode ocorrer pode ser uma coisa muito útil. Por Exemplo:

```js
var a = {
	num: 21,
	toString: function() { return String( this.num * 2 ); }
};

parseInt( a ); // 42
```

O fato de que `parseInt (...)` forçe a coerção de seu valor para um `string` para realizar um parse é bastante sensato. Se você passar lixo, e você receber lixo de volta, não culpe a lata de lixo -- ela só fez seu trabalho fielmente.

Então, se você passar um valor como `Infinity` (o resultado de `1 / 0` obviamente), que tipo de representação de `string` você faria mais sentido para essa coerção? Apenas duas escolhas racionais vêm à mente: `"Infinity"` and `"∞"`. O JS escolhe `"Infinity"`. E sou grato por ele escolher isso.

Eu acho que é uma coisa boa que **todos os valores** em JS tenham algum tipo de representação de `string` padrão, assim eles não são misteriosas caixas preta que nós não podemos debugar e pensar sobre.

Agora, e sobre caracteres base-19? Obviamente, completamente falso e artificial. Nenhum programa JS real usa base-19. É um absurdo. Mas, de novo, vamos curtir o ridículo. Em base-19, os caracteres numéricos válidos são `0` - `9` e `a` - `i` (case insensitive).

Então, de volta para nosso exemplo `parseInt( 1/0, 19 )`. Isso é essencialmente `parseInt( "Infinity", 19 )`. Como ele irá parsear? O primeiro caractere é o `"I"`, no qual é valor `18` na boba base-19. O segundo caractere `"n"` não está no conjuntos de caracteres válidos, e como tal, o parse simplesmente pára, assim como quando ele cruzar com `"p"` em `"42px"`.

O resultado? `18`. Exatamente como ele, sensatamente, deve ser. Os comportamentos envolvidos para nos trazer até aqui, e não para um próprio erro `Infinity`, são **muito importantes** para o JS, e não devem ser descartados tão facilmente.

Outros examplos desse comportamento com `parseInt(..)` que podem ser surpreendentes mas são bastante sensatos incluem:

```js
parseInt( 0.000008 );		// 0   ("0" de "0.000008")
parseInt( 0.0000008 );		// 8   ("8" de "8e-7")
parseInt( false, 16 );		// 250 ("fa" de "false")
parseInt( parseInt, 16 );	// 15  ("f" de "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```

Na verdade `parseInt(..)` é bem previsível e consistente em seu comportamento. Se voce usa-lo corretamente, você terá resultados sensatos. Se você usa-lo incorretamente, o resultado maluco que você terá não é culpa do JavaScript.

### Explicitamente: * --> Boolean

Agora, vamos examinar a coerção de qualquer outro valor não `boolean` para um `boolean`.

Assim como com `String(..)` e `Number(..)` acima, `Boolean(..)` (sem o `new`, claro!) é uma forma explícita de forçar a coerção `ToBoolean`:

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

Boolean( a ); // true
Boolean( b ); // true
Boolean( c ); // true

Boolean( d ); // false
Boolean( e ); // false
Boolean( f ); // false
Boolean( g ); // false
```

Enquanto `Boolean(..)` é claramente explícita, ela não é tão comum ou idiomática.

Assim como o operador unário `+` faz a coerção de um valor para um `number` (veja acima), o operador unário de negação `!` faz a coerção explicitamente de um valor para um `boolean`. O *problema* é que ele também inverte o valor de verdadeiro para falso ou vice versa. Então a forma mais comum em que desenvolvedores JS fazem a coerção explícita para `boolean` é usando o operador de negação duplo `!!`, porque o segundo `!` vai inverter a paridade de volta à original:

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

!!a;	// true
!!b;	// true
!!c;	// true

!!d;	// false
!!e;	// false
!!f;	// false
!!g;	// false
```

Qualquer uma dessas coerções `ToBoolean` podem acontecer *implicitamente* sem o `Boolean(..)` ou `!!`, se usado em um contexto `boolean` assim como uma declaração `if(..) ..`. Mas o objetivo é forçar explicitamente o valor para um `boolean` para deixar claro que a coerção `ToBoolean` é intencional.

Outro caso de uso para coerção explícita `ToBoolean` é se você quer forçar uma coerção de valor `true`/`false` em uma serialização JSON de uma estrutura de dados:

```js
var a = [
	1,
	function(){ /*..*/ },
	2,
	function(){ /*..*/ }
];

JSON.stringify( a ); // "[1,null,2,null]"

JSON.stringify( a, function(key,val){
	if (typeof val == "function") {
		// força a coerção `ToBoolean` da função
		return !!val;
	}
	else {
		return val;
	}
} );
// "[1,true,2,true]"
```

Se você veio para o JavaScript do Java, você deve reconhecer essa linguagem:

```js
var a = 42;

var b = a ? true : false;
```

O operador ternário `? :` vai testar `a` para verdadeiro, e baseado nesse teste atribuirá `true` ou `false` para `b`, em conformidade.

Nessa superfície, essa linguagem é uma forma *explícita* de coerção do tipo `ToBoolean`, uma vez que é óbvio que apenas `true` ou `false` saem da operação.

Porém, há uma coerção *implícita* oculta, aquela expressão `a` deve primeiro sofrer a coerção para `boolean` para executar o teste de verdade. Eu chamaria essa linguagem de "explicitamente implícita". Além disso, eu sugiro que **você evite essa linguagem completamente** no JavaScript. Ela não oferece benefícios reais, e pior, mascara algo que não é.

`Boolean(a)` e `!!a` são de longe melhores as opções para coerção *explícita*.

## Coerção Implícita

Coerção *implícita* se refere à tipos de conversões que são ocultas, com efeitos colaterais não óbvios que implicitamente ocorrem por outras ações. Em outras palavras, *coerções implicitas* são qualquer tipo de conversões que não são óbvias (para você).

Enquanto está claro qual é o objetivo de coerção *explícita* (tornar o código explícito e compreensível), pode ser *muito* óbvio que coerção *implícita* tenha o objetivo oposto: tornar o código mais difícil de se entender.

Confiar de olhos fechados, acredito que é aí que grande parte da raiva de coerções vêm. A maioria das reclamações sobre "coerções JavaScript" têm, na verdade, como alvo (eles percebendo ou não) coerções *implícitas*.

**Observação:** Douglas Crockford, autor de *"JavaScript: The Good Parts"*, afirmou em muitas palestras de conferências e artigos que coerção JavaScript deve ser evitada. Mas o que ele pareceu falar é que coerção *implícita* é ruim (na opnião dele). Porém, se você ler seu próprio código, você irá achar muitos exemplos de coerção, ambas *implícita* e *explícita*! Na verdade, a raiva dele parece ser primeiramente destinada para a operação `==`, mas você verá nesse capítulo, essa é apenas uma parte do mecanismo de coerção.

Então, a **coerção implícita é** maligna? Ela é perigosa? É uma falha no design do JavaScript? Nós devemos evitá-la a todo custo?

Aposto que a maioria de vocês, leitores, está inclinada a torcer com entusiasmo, "Sim!"

**Não tão rápido**. Me dê ouvidos.

Vamos assumir uma perspectiva diferente do que é coerção *implícita*, e pode ser, do que apenas que é "o oposto do bom tipo explícito de coerção". Isso é muito estreiro e perde uma nuance importante.

Vamos definir o objetivo de coerção *implícita* como: reduzir a verbosidade, boilerplate, e/ou detalhes de implentação desnecessários que encobre nosso código com ruído que nos distrai da intenção mais importante.
